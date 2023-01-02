# Updating your `mod.json` file
## Your mod.json file will be outdated, so it's up to us to keep it updated.
<br/>

Set the `packageVersion` to whatever version of Beat Saber you're developing your mod for.  
<br/>

Set the `libbeatsaber-hook_2_3_2.so` to the latest Beatsaber-hook version. You can check the latest version by going to the [latest releases](https://github.com/sc2ad/beatsaber-hook/releases/latest) and looking at the version.  
![](/images/beatsaberhook-version.png)  
In my case, I will edit the `libbeatsaber-hook_2_3_2.so` to say `libbeatsaber-hook_3_14_0.so`
<br/>

Also edit your `qpm.json` file. The `versionRange` for beatsaber-hook should be the latest version. In my case, I will edit it to say `^3.14.0`.  
<br/>

## At this point, we are ready to start making our mod!
[Next](./coding-your-mod.md)