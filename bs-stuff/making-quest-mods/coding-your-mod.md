# [Home](https://cgray1234.github.io/index)  
<br/>

<style>
    * {
        font-family: "Teko";
        src: url(teko-medium.otf);
    }
</style>

# Coding your mod
## It's time for the moment you've been waiting for, making the mod! This mod will be a simple mod that changes the text of the main menu buttons (solo button, online button, etc.) We will be using codegen, config-utils, custom-types, and QuestUI.  
<br/>

First, we need to add our dependencies using QPM-Rust. In a new terminal, run this code:  
`qpm-rust dependency add codegen`  
`qpm-rust dependency add config-utils`  
`qpm-rust dependency add questui`  
`qpm-rust dependency add custom-types`  
In case you need to use an older version of any of these core mods, just add `-v ^version`  
<br/>

Now run `qpm-rust restore` to download these dependencies. Codegen takes a while to download because it's a lot of files, so you need to be patient with this process.  
<br/>

# Now, we can move onto making our config.
Make a new file in the "`include`" folder. You can name this whatever you want, I'll be naming it config.hpp. This must be a .hpp file.  
<br>

To make our config, write the following code:
```
#pragma once // Always put this at the top of all of your header files (.hpp files)

// Include the files we need to use
#include "config-utils/shared/config-utils.hpp"
#include "UnityEngine/Color.hpp" // Since our config values will be using color, we must include that file

// Makes our config
DECLARE_CONFIG(ModConfig, // The config name that will be used in our files
    CONFIG_VALUE(SoloButtonText, std::string, "Solo Button Text", "Solo");
    CONFIG_VALUE(SoloButtonTextColor, UnityEngine::Color, "Solo Button Text Color", UnityEngine::Color::get_white());

    CONFIG_VALUE(OnlineButtonText, std::string, "Online Button Text", "Online");
    CONFIG_VALUE(OnlineButtonTextColor, UnityEngine::Color, "Online Button Text Color", UnityEngine::Color::get_white());

    CONFIG_VALUE(CampaignButtonText, std::string, "Campaign Button Text", "Campaign");
    CONFIG_VALUE(CampaignButtonTextColor, UnityEngine::Color, "Campaign Button Text Color", UnityEngine::Color::get_white());

    CONFIG_VALUE(PartyButtonText, std::string, "Party Button Text", "Party");
    CONFIG_VALUE(PartyButtonTextColor, UnityEngine::Color, "Party Button Text Color", UnityEngine::Color::get_white());

    // Our config values are (Name, type, Json name, default value)
)
```  
<br/>

Almost done with our config, all we need to do now is register it in our `main.cpp` file.  
<br/>
To do this, put the following code in the `load()` part of our `main.cpp` file. It should look like this:
```
#include "config.hpp" // Put this at the top of your file

extern "C" void load() {
    il2cpp_functions::Init();

    getModConfig().Init(modInfo);

    getLogger().info("Installing hooks...");
    // Install our hooks (none defined yet)
    getLogger().info("Installed all hooks!");
}
```


Now our config is complete and ready to be used in the mod!  
<br/>

# Next, we can start making our hooks.
Our hooks are always in our `main.cpp` file in the `src` folder.  
<br/>

We need to put our hooks in the spot shown below:  
![](/images/hooks-go-here.png)  
<br/>

To make the main menu buttons change their title and color, we must input the following code:
```
#include "config.hpp"

#include "GlobalNamespace/MainMenuViewController.hpp"
#include "HMUI/CurvedTextMeshPro.hpp"
#include "UnityEngine/GameObject.hpp"

MAKE_HOOK_MATCH(MainMenuViewController_DidActivate, &GlobalNamespace::MainMenuViewController::DidActivate, void, GlobalNamespace::MainMenuViewController* self, bool firstActivation, bool addedToHierarchy, bool screenSystemEnabling) {
    MainMenuViewController_DidActivate(self, firstActivation, addedToHierarchy, screenSystemEnabling); // Put this at the very top, or very bottom if you want to change arguments

    UnityEngine::GameObject* soloGameObject = self->soloButton->get_gameObject();
    HMUI::CurvedTextMeshPro* soloButtonText = soloGameObject->GetComponentInChildren<HMUI::CurvedTextMeshPro*>();

    soloButtonText->SetText(getModConfig().SoloButtonText.GetValue());
    soloButtonText->set_color(getModConfig().SoloButtonTextColor.GetValue());


    UnityEngine::GameObject* onlineGameObject = self->multiplayerButton->get_gameObject();
    HMUI::CurvedTextMeshPro* onlineButtonText = onlineGameObject->GetComponentInChildren<HMUI::CurvedTextMeshPro*>();

    onlineButtonText->SetText(getModConfig().OnlineButtonText.GetValue());
    onlineButtonText->set_color(getModConfig().OnlineButtonTextColor.GetValue());


    UnityEngine::GameObject* campaignGameObject = self->campaignButton->get_gameObject();
    HMUI::CurvedTextMeshPro* campaignButtonText = campaignGameObject->GetComponentInChildren<HMUI::CurvedTextMeshPro*>();

    campaignButtonText->SetText(getModConfig().CampaignButtonText.GetValue());
    campaignButtonText->set_color(getModConfig().CampaignButtonTextColor.GetValue());


    UnityEngine::GameObject* partyGameObject = self->partyButton->get_gameObject();
    HMUI::CurvedTextMeshPro* partyButtonText = partyGameObject->GetComponentInChildren<HMUI::CurvedTextMeshPro*>();

    partyButtonText->SetText(getModConfig().PartyButtonText.GetValue());
    partyButtonText->set_color(getModConfig().PartyButtonTextColor.GetValue());
}
```  
<br/>

Yeah, it's a lot, which is why I've provided it here for you so you don't have to type it all out :)  
<br/>

After that, we need to install our hooks. To install the hooks, add the following code into the `load()` part in `main.cpp`
```
// Called later on in the game loading - a good time to install function hooks
extern "C" void load() {
    il2cpp_functions::Init();

    getLogger().info("Installing hooks...");

    INSTALL_HOOK(getLogger(), MainMenuViewController_DidActivate); // Installs our hooks

    getLogger().info("Installed all hooks!");
}
```  
<br/>

# We're almost done! Now, we need to make our UI.
To make our UI, we will be using custom-types and QuestUI. To get started, make a file called `ViewController.hpp` in our include folder.

```
#pragma once

#include "custom-types/shared/macros.hpp"
#include "HMUI/ViewController.hpp"

// TestMod is our namespace, TestModViewController is our type name
DECLARE_CLASS_CODEGEN(TestMod, TestModViewController, HMUI::ViewController,

    DECLARE_OVERRIDE_METHOD(void, DidActivate, il2cpp_utils::FindMethodUnsafe("HMUI", "ViewController", "DidActivate", 3), bool firstActivation, bool addedToHierarchy, bool screenSystemEnabling);
)
```  
<br/>

Once we're done with that, make a file in your `src` folder called `ViewController.cpp`. This is the file where we make our UI.  
<br/>

Put the following code into your `ViewController.cpp` file:
```
#include "UI/ViewController.hpp"
#include "config.hpp"

#include "questui/shared/BeatSaberUI.hpp"
#include "questui/shared/QuestUI.hpp"

#include "UnityEngine/GameObject.hpp"
#include "UnityEngine/Color.hpp"

DEFINE_TYPE(TestMod, TestModViewController); // Defines our type so we can use it in the code below

void TestMod::TestModViewController::DidActivate(bool firstActivation, bool addedToHierarchy, bool screenSystemEnabling) {
    if (firstActivation) { // Call this so it will only make our UI once
        UnityEngine::GameObject* container = QuestUI::BeatSaberUI::CreateScrollableSettingsContainer(get_transform()); // Makes the scrollable container to put our UI elements (buttons, switches, color pickers, etc.) in

        // Creates text that says "Solo Button", parents it to the container, and aligns it in the center
        QuestUI::BeatSaberUI::CreateText(container->get_transform(), "Solo Button")->set_alignment(TMPro::TextAlignmentOptions::Center);

        // Creates an editable string setting in the UI that gets the current string value in our mod config, and applies the string value when we interact with it (use the = lambda for this)
        QuestUI::BeatSaberUI::CreateStringSetting(container->get_transform(), "Solo Button Text", getModConfig().SoloButtonText.GetValue(), 
            [=](std::string value) {
                getModConfig().SoloButtonText.SetValue(value);
            }
        );
        // Creates a color picker that says "Solo Button Text Color", gets the value in our mod config, and applies the changes when we're done interacting with it (again, use the = lambda)
        QuestUI::BeatSaberUI::CreateColorPicker(container->get_transform(), "Solo Button Text Color", getModConfig().SoloButtonTextColor.GetValue(),
            [=](UnityEngine::Color value) {
                getModConfig().SoloButtonTextColor.SetValue(value);
            }
        );


        QuestUI::BeatSaberUI::CreateText(container->get_transform(), "Online Button")->set_alignment(TMPro::TextAlignmentOptions::Center);

        QuestUI::BeatSaberUI::CreateStringSetting(container->get_transform(), "Online Button Text", getModConfig().OnlineButtonText.GetValue(), 
            [=](std::string value) {
                getModConfig().OnlineButtonText.SetValue(value);
            }
        );

        QuestUI::BeatSaberUI::CreateColorPicker(container->get_transform(), "Online Button Text Color", getModConfig().OnlineButtonTextColor.GetValue(),
            [=](UnityEngine::Color value) {
                getModConfig().OnlineButtonTextColor.SetValue(value);
            }
        );
        

        QuestUI::BeatSaberUI::CreateText(container->get_transform(), "Campaign Button")->set_alignment(TMPro::TextAlignmentOptions::Center);

        QuestUI::BeatSaberUI::CreateStringSetting(container->get_transform(), "Campaign Button Text", getModConfig().CampaignButtonText.GetValue(), 
            [=](std::string value) {
                getModConfig().CampaignButtonText.SetValue(value);
            }
        );

        QuestUI::BeatSaberUI::CreateColorPicker(container->get_transform(), "Campaign Button Text", getModConfig().CampaignButtonTextColor.GetValue(),
            [=](UnityEngine::Color value) {
                getModConfig().CampaignButtonTextColor.SetValue(value);
            }
        );


        QuestUI::BeatSaberUI::CreateText(container->get_transform(), "Party Button")->set_alignment(TMPro::TextAlignmentOptions::Center);

        QuestUI::BeatSaberUI::CreateStringSetting(container->get_transform(), "Party Button Text", getModConfig().PartyButtonText.GetValue(), 
            [=](std::string value) {
                getModConfig().PartyButtonText.SetValue(value);
            }
        );

        QuestUI::BeatSaberUI::CreateColorPicker(container->get_transform(), "Solo Button Text", getModConfig().PartyButtonTextColor.GetValue(),
            [=](UnityEngine::Color value) {
                getModConfig().PartyButtonTextColor.SetValue(value);
            }
        );
    }
}
```
<br/>

# We're almost done making our UI! Now, we have to register it into our game.
This next step is SUPER simple. All we need to do is register our UI in our `main.cpp` file. To do that, we must add the following code into our `main.cpp` file
```
// Add this at the top of your main.cpp file
#include "questui/shared/QuestUI.hpp"
#include "ViewController.hpp"
```
Now, we need to add this into our `load()` function
```
extern "C" void load() {
    il2cpp_functions::Init();

    getModConfig().Init(modInfo);

    QuestUI::Init();
    QuestUI::Register::RegisterAllModSettingsViewController<TestMod::TestModViewController*>(modInfo, "Test Mod");

    getLogger().info("Installing hooks...");
    INSTALL_HOOK(getLogger(), MainMenuViewController_DidActivate);
    getLogger().info("Installed all hooks!");
}
```  
<br/>

# We now have our UI set up!
# Our mod should be done, run copy.ps1 to test our mod!
[Next](./building-your-qmod.md)