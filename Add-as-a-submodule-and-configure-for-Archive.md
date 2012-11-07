Adding awesome projects like SDWebImage as submodules to your main git project is a great way to keep your builds fresh and sturdy.

Every git repo is a new beast because folks offer .pbxproj files with varying levels of effort put into them.

Adding SDWebImage as a submodule was relatively easy but I did stumble a bit so here are the steps:

* cd yourXCodeProject/
* git submodule add https://github.com/rs/SDWebImage.git
* cd yourXCodeProject/SDWebImage
* Open the .pbxproj file for SDWebImage and run a build, doesn't matter which version you run against iOS5 or iOS6
* Once it is successful, close the .pbxproj file
* Open the .pbxproj file for YourXCodeProject
* Drag & Drop the SDWebImage's .pbxproj file under YourXCodeProject
* If you want to rebuild SDWebImage automagically if you ever decide to assimilate some changes in the future from git ... then goto YourXCodeProject > Targets > YourXCodeProject > Build Phase > Target Dependencies > + > SDWebImage or SDWebImage ARC (whatever makes sense for you) ... this also has the added benefit of SDWebImage.framework becoming available to you on each such build w/o having to download it from somewhere.
* YourXCodeProject > Targets > YourXCodeProject > Build Phase > Link Bindary With Libraries > + > Add libSDWebImage.a (ARC) or whatever makes sense for you. Without this, you will face runtime crashes stating that the method you want to use doesn't really exist and is an illegal selector etc.
* YourXCodeProject > Targets > YourXCodeProject > Build Phase > Link Bindary With Libraries > + > Add ImageIO.framework if it makes sense for you (iOS6 - see instructions in SDWebImage's readme on github).
* And without this final step you won't be able to Archive: YourXCodeProject > Targets > YourXCodeProject > Build Settings > Header Search Paths > + > "$(SRCROOT)/SDWebImage"

All done!

Do yourself a favor and get something like SourceTree (free UI on top of git from Atlassian) or at least listen to git when it tells you that your submodules have updates available. Overall the power of submodules is in the fact that you get a chance to notice on your own that updates are available and them make a call on what you want to assimilate and what is better left alone. If the awesome projects on GitHub suffer from one weakness ... its their lines of communication back to their users about updates/bug-fixes etc. With submodules you can be smarter, stay informed, and use more & more components to build out you next great app.