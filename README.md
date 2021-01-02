# gdx-liftoff
A modern setup tool for libGDX Gradle projects, forked from czyzby/gdx-setup

![Screenshot of gdx-liftoff](https://i.imgur.com/XtLhaK2.png)

If you've used libGDX for even a short time, you've probably used the official `gdx-setup.jar` made by the libGDX team. It's seen
major improvements lately, but it still lags behind in some areas -- it places assets in different locations depending on how you
initially configured your project, and it is very far behind the times on its default LWJGL version and on third-party extensions.
The official setup may transition to a web-based tool soon, but any user of the Internet can recall times when
formerly-reliable services went offline or had outages. This project provides another alternative setup tool based on
[SquidSetup](https://github.com/tommyettinger/SquidSetup), but removing the close ties to the SquidLib libraries to make it more general-use. Using SquidSetup's
code, which is built on czyzby's code, gives us working projects that use Gradle 6.7.1, the same as the official setup, but ahead of 4.0.2 for czyzby's gdx-setup.
Currently, gdx-liftoff projects depend on libGDX 1.9.13 by default, and allow using snapshots as well.

Projects default to using LWJGL3 instead of LWJGL2 (the old 'desktop' platform), since code tends to be very similar between the two, but LWJGL3 generally offers
more features. This code is tested for compatibility with GWT, including the various changes that Gradle needs with this version. It is sometimes tested on Android,
but Android Studio is often incompatible with recent Gradle releases, and Android certainly doesn't support Java 13 or higher features across the board. Issues
with iOS and RoboVM will have to be addressed by someone sending a pull request, because I can't reproduce any iOS issues without an iOS device.

The current version of gdx-liftoff uses LWJGL3 internally; 1.9.10.5 used LWJGL2 in an attempt to be compatible with libGDX's TexturePacker, but there seem to be
more compatibility issues with LWJGL2, maybe since it hasn't been updated in 5+ years, than with LWJGL3. This perhaps validates the decision to default to LWJGL3
for new projects generated by gdx-liftoff.

## Usage

  - Get the latest `gdx-liftoff.jar` from the [Releases tab](https://github.com/tommyettinger/gdx-liftoff/releases) of this project.
    - If you have an older gdx-liftoff.jar, or you aren't sure when it was released, getting the current latest is a good idea.
  - Regardless of what platforms you intend to target, make sure the steps
    [described by the libGDX wiki here](https://github.com/libgdx/libgdx/wiki/Setting-up-your-Development-Environment-%28Eclipse%2C-Intellij-IDEA%2C-NetBeans%29)
    are taken care of.
  - Run the JAR. Plug in whatever options you see fit:
    - For the Platforms tab, LWJGL3 starts enabled by default (it works on all desktop/laptop platforms), and you can
      manually check other platforms to match your needs. 
      If you target iOS, it will only build on a MacOS machine. Downloading iOS (and/or HTML) dependencies can take some
      time, so just check the platforms you want to target. You can re-run the setup, make a new project with the same 
      settings (in a different folder), and copy in the existing code if you want to quickly change platforms.
      - Desktop and/or LWJGL3 should usually be checked, so you can test on the same computer you develop on.
        - LWJGL3 is almost the same as Desktop, but because it has better support for new hardware
          (such as high-DPI displays), it should probably be preferred. It also allows multiple windows and drag+drop.
          - LWJGL3 itself supports Linux on arm32 and arm64 hardware, and the current 1.9.13 release of libGDX also
            supports ARM on desktop platforms.
        - Desktop should mostly be preferred if you need to also depend on gdx-tools, such as if you need to run the
          texture packer at runtime. Some machines have issues with an inconsistent or very high framerate with LWJGL3,
          and using the "Legacy" desktop can fix that. This platform can also be less compatible with some JDKs, and
          distributing a JDK from [AdoptOpenJDK](https://adoptopenjdk.net/) usually fixes that.
          - The framerate limit problem is currently solved with both Desktop and LWJGL3, as long as you use libGDX
            1.9.12 or higher.
          - The "less compatible" JDK issue manifests as a crash immediately at startup, with several warnings printed
            (which themselves don't matter), and a line right after them mentioning `ld.so` or a linking error. If you
            encounter this, switch to AdoptOpenJDK or LWJGL3.
      - iOS should probably not be checked if you aren't running MacOS and don't intend to later build an iOS
        app on a Mac. It needs some large dependencies to be downloaded when you first import the project.
        - If you have a Mac that is set up for iOS development, please try to generate any project and see if it gets
          made correctly! We've had some [good feedback](https://github.com/tommyettinger/gdx-liftoff/issues/28) on iOS
          projects, but any extra usage info would help ensure that liftoff is ready for any libGDX usage. It isn't a
          typical usage for a [GitHub Issue](https://github.com/tommyettinger/gdx-liftoff/issues), but sending any
          feedback as an issue, whether it's "iOS projects work for me" or "iOS support is broken" would really help.
      - Android should only be checked if you've set up your computer for Android development. Since gdx-liftoff uses
        Gradle 6.7.1, having an Android project present shouldn't interfere with other platforms or IDE integration, as
        long as your IDE supports Gradle 6.7.1 (Android Studio may in its most recent versions, but IntelliJ IDEA (and
        Eclipse with Buildship) should automatically).
        - Having an Android module in a larger project changes some of IDEA's features, including disabling hot-swap.
          Some libGDX developers take the approach of having a separate Android-only project, keeping desktop platforms
          completely disconnected from Android. This also lets the assets be different, so it has other advantages.
        - If `Java version` is set to 8 on the Advanced tab, then gdx-liftoff sets up the Android configuration to use
          core library desugaring and other Java-8-related features, allowing a large subset of Java 8 language
          features, and the standard library in JDK 8, to be used across most platforms (not iOS, though).
      - HTML is a more-involved target, with some perfectly-normal code on all other platforms acting completely
        different on HTML due to the tool used, Google Web Toolkit (GWT). It's almost always possible to work around
        these differences and make things like random seeds act the same on all platforms, but it takes work. Mostly,
        you need to be careful with the `long` and `int` number types, and relates to `int` not overflowing as it
        would on desktop, and `long` not being visible to reflection. See [this small guide to GWT](https://github.com/libgdx/libgdx/wiki/HTML5-Backend-and-GWT-Specifics)
        for more. It's very likely that you won't notice any difference unless you try to make behavior identical on GWT
        and other platforms, and even then there may be nothing apparent.
        - GWT 2.9.0 is available but doesn't integrate with libGDX by default; there's a third-party [replacement to the
          official GWT backend](https://github.com/tommyettinger/gdx-backends#19120) that supports it with libGDX
          1.9.11. Using GWT 2.9.0 allows Java 11's `var` keyword to be used, plus other Java 11 features, but doesn't
          change much of what's available from the standard library.
    - For dependencies, you don't need libGDX checked (the tool is ready to download libGDX and set it as a
      dependency in all cases).
    - There are options to add language support for non-Java JVM languages; of these, Kotlin is the best-supported.
      - The HTML target won't work with non-Java languages, and others targets may be questionable.
        - Kotlin should work well on Android, Desktop (LWJGL2) and LWJGL3, and will probably work well on iOS.
          - You can see some extra ways to use Kotlin as the Gradle build language in Quillraven's
            [Dark-Matter](https://github.com/Quillraven/Dark-Matter) repo; it also uses Kotlin launchers.
          - Kotlin launchers should be used cautiously; on iOS in particular there have been cases where they caused
            mysterious, hair-pulling issues despite working on Android and desktop. You usually edit launchers rarely,
            so if they're working in Java, you generally don't need to change them to Kotlin.
          - Some third-party extensions only work with Kotlin, lke the KTX libraries.
        - Scala and Groovy should definitely work on Desktop and LWJGL3, and may work on Android and iOS.
        - Clojure may technically work on Android but is usually incredibly slow without extra steps for Android
          compatibility; it's doubtful if it would work on iOS. You probably shouldn't use Gradle to build Clojure
          projects anyway; it has its own (excellent) project manager `lein` and a simple built-in manager.
    - In the Templates tab, you can select various sets of starting code that gdx-liftoff will generate in your new
      project. Classic will show a white screen with a pixel-style face when you run, so it can be good to verify that
      a project works, while ApplicationAdapter is probably the easiest to bring an existing game into.
    - In Advanced, you can set the libGDX version (it defaults to 1.9.13, but can be set lower or higher) and
      various other versions, including the default Java compatibility. Typically, `Java version` is the minimum across
      all platforms, and should be 7 or more (8 is generally safe). You can set `Desktop Java version` to any version at
      least equal to `Java version`, and similarly for `Server Java version`; these only affect the Desktop/LWJGL3 and
      Server modules, respectively. You can set `Java version` to as high as 15 if you have Java 15 installed, or
      similarly for Java 11, 12, 13, or 14, but it will require users to also have Java of that version, or for you to
      distribute a JRE of the appropriate version with your game.
      - Distributing Java 14 or 15 is much easier now thanks to Beryx'
        "[Badass Runtime Plugin](https://github.com/raeleus/skin-composer/wiki/Deploying-libGDX-with-jpackage-and-Badass-Runtime),"
        which may be included in future versions to help ease the process of releasing a game.
  - Click generate, and very soon a window should pop up with instructions for what to do.
    - Generation is very fast here, relative to gdx-setup, because it doesn't run Gradle tasks at this point. When you
      see `SETUP COMPLETE` in green, the build is done; at the time you import the generated `build.gradle` project
      file, some tasks will run.
    
Now you'll have a project all set up with a sample. In IntelliJ IDEA or Android Studio, you can choose to open the
`build.gradle` file and select "Open as Project" to get started. In Eclipse or Netbeans, the process should be similar;
see [libGDX's documentation](https://libgdx.badlogicgames.com/documentation/gettingstarted/Importing%20into%20IDE.html).

  - The way to run a game project that's probably the most reliable is to use Gradle tasks
    to do any part of the build/run process. The simplest way to do this is in the IDE itself,
    via `View -> Tool Windows -> Gradle`, and selecting tasks to perform, such as
    `lwjgl3 -> Tasks -> application -> run.` If you try to run a specific class' `main()`
    method, you may encounter strange issues, but this shouldn't happen with Gradle tasks.
    - To run a `main()` method, you usually need to set a run configuration so the working folder is `assets`.
      The location of the `assets` folder is different here than with the official setup; it's at the same depth as
      `core` or `lwjgl3`, and doesn't change across configurations like with the official setup.
  - If you had the LWJGL3 (or Desktop) option checked in the setup, and you chose a non-empty
    template in the Templates tab, you can run the LWJGL3 or Desktop module right away.
    - You can build a runnable jar that includes all it needs to run using
      `lwjgl3 -> Tasks -> build -> jar`; this jar will be in `lwjgl3/build/libs/` when it finishes.
      Note: this is the command-line option `gradlew lwjgl3:jar`, not the `dist` command
      used by the official setup jar. Substitute `desktop` where `lwjgl3` is if you use the legacy
      LWJGL2 version.
  - If you had the Android option checked in the setup and have a non-empty template,
    you can try to run the Android module on an emulator or a connected Android device.
  - If you had the GWT option checked in the setup and have a non-empty template,
    you can go through the slightly slow, but simple, build for GWT, probably using the `superDev`
    task for the `gwt` module, or also possibly the `dist` task in that module.
      - GWT builds have gotten much faster with Gradle 6.7.1 and some adjustments to configuration, so
        if you were avoiding GWT builds because of slow compile times, you might want to try again.
  - If you had the iOS option checked in the setup, you're running Mac OS X,
    and you have followed all the steps for iOS development with libGDX, maybe you can run
    an iOS task? I can't try myself without a Mac or iOS device, so if you can get this to
    work, posting an issue with any info for other iOS targeters would be greatly appreciated.
    - Agreeing with libGDX, MOE is no longer supported. Although there are "MOE Community" builds, they come with
      a breaking change disclaimer that appears likely to break libGDX with MOE Community 1.6.0. You can use
      gdx-liftoff 1.9.10.9 if you desperately need MOE for some reason.
  - All builds currently use Gradle 6.7.1 with the "api/implementation/compile fiasco" resolved. Adding dependencies
    will use the `api` keyword instead of the `compile` keyword it used in earlier versions. All modules use the
    `java-library` plugin, which enables the `api` keyword for dependencies.
  - You may need to refresh the Gradle project after the initial import if some dependencies timed-out;
    JitPack dependencies in particular may take up to 15 minutes to become available if you're using any of those,
    like SquidLib. In IntelliJ IDEA, the `Reimport all Gradle projects` button is a pair of circling arrows in the
    Gradle tool window, which can be opened with `View -> Tool Windows -> Gradle`.
  - Out of an abundance of caution, [the dependency impersonation issue reported here by Márton
    Braun](https://blog.autsoft.hu/a-confusing-dependency/) is handled the way he handled it, by putting
    `jcenter()` last in the repositories lists. I don't know if any other tools have done the same, but it's
    an easy fix, and I encourage them to do so.
    
## Known Issues

  - MacOS does not like the legacy desktop apps, showing all sorts of visual glitches.
    It seems to work fine with LWJGL3, in part because that platform had special attention
    paid to it so the `gradlew lwjgl3:run` command can work at all on MacOS.
  - Android hasn't been tested enough, and the generated manifest is probably not very good.
    - You can modify the manifest, and probably need to do so if you want to submit an app to the Play store.
    - Android Studio should have better support for recent Gradle versions if you use a beta release. 
    - Android Studio should be at least version 4.0, but 4.1 is untested so far.

## Credits

Huge thanks to [czyzby](https://github.com/czyzby) for writing the original tool this is based on, as well as much of
the code gdx-liftoff depends on. Thanks also to Raymond Buckley for making the
[Particle Park skin for scene2d.ui](https://ray3k.wordpress.com/particle-park-ui-skin-for-scene2d-ui/), which was
adapted to be the skin added to new projects (if you choose the "Generate UI Assets" option in the Advanced tab). More
thanks to "Accademia di Belle Arti di Urbino and students of MA course of Visual design" for making the Titillium Web
font that the skin uses (SIL OFL license). Thanks to anyone who's contributed code to gdx-liftoff (
[Mr00Anderson](https://github.com/Mr00Anderson),
[lyze237](https://github.com/lyze237), [metaphore](https://github.com/metaphore), and
[payne911](https://github.com/payne911), so far)! And of course, thanks to all the early adopters, for putting up with
any partially-working releases I churned out.

Good luck, and I hope you make something great!

