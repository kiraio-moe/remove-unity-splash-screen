# Remove Unity Splash Screen

## Intoduction

Do you hate this logo appears when you are going to play games? Bother? For some people, yes.
This time I will share one of the things I learned when I Reverse Engineering the Unity game, removing the Unity splash screen XD.

I haven't tried on some platforms because I haven't installed the dependencies needed to build games on those platforms in Unity, maybe I'll try them in the future.  
But if you manage to remove the Unity splash screen using another version of Unity, please make a [Pull Request](https://github.com/kiraio-moe/remove-unity-splash-screen "Pull Request") or contact me on [my social media](https://github.com/kiraio-moe) and let me know the details so I can get it straight away update this article.  

Without further ado, let's do it!

### Tested

Tested and work on games built with the following versions of Unity:

#### Unity 2020.3.39.26224 (LTS)

- PC/Windows x86_64 (Mono)
- Android (Mono, IL2CPP)

#### Unity 2019.4.12f1 (LTS)

- Android (Mono, IL2CPP)

## Prerequisite

Install the following tools before proceed to [Step-by-step section](#step-by-step):

### General

- [Unity Asset Bundle Extractor (UABE)](https://github.com/SeriousCache/UABE/releases "Unity Asset Bundle Extractor")
- [HxD Hex Editor](https://mh-nexus.de/en/hxd/ "HxD Hex Editor")

### Platform Specific

Install the following tools if you wanna work with another platform:

#### Android

- [APKTool GUI](https://github.com/AndnixSH/APKToolGUI/releases "APKTool GUI")

#### WebGL

- Coming soon... I guess?

## Step-by-step

I don't know how games built with IL2CPP looks like on PC platforms. If you don't mind, please create a Dummy project and build with IL2CPP scripting backend and send the build to me via email: <itsyuukunz@gmail.com> . I'll try playing with it ;)

### PC (Mono)

- Prepare the Unity game that has been built.  
![Target Unity game](./img/IMG_PC_Mono_01.png "Target Unity game")
- Go to the ``Game Name_Data`` folder and backup the ``globalgamemanagers`` file in case of errors.
- Open the ``globalgamemanagers`` file using Unity Asset Bundle Extractor (UABE).
- Click ``globalgamemanagers (Assets)`` in Files and Components. Select ``Unnamed asset`` which is of type **``PlayerSettings``** then click **View Data** and...  
![View Player Settings](./img/IMG_PC_Mono_02.png "View PlayerSettings")
- BOOM! Error................  
![View Player Setings Error](./img/IMG_PC_Mono_03.png "View PlayerSettings Error")  
This happens in games using Unity version >= 2020 where it seems Unity has encrypted (probably) the PlayerSettings part but not the other part. Whereas when I try to open the **PlayerSettings** section of the game built using Unity 2019, UABE can still access it.
- We will fix the problems in the Unity 2020 version later. For now, let's focus on games built using Unity <= 2019. As you can see, we can access PlayerSettings. If we change the value of ``m_ShowUnitySplashScreen`` to ``false`` (by double clicking) then the Splash Screen will not appear when starting the game (the **Made with Unity** logo will also not appear automatically) and as you can guess, if we change the value of ``m_ShowUnitySplashLogo`` to ``false`` then the **Made with Unity** logo will not appear.  
![Accessing PlayerSettings Unity 2019](./img/IMG_PC_Mono_04.png "Accessing PlayerSettings Unity 2019")
But this (just removing the **Made with Unity** logo) will not work if the **Made with Unity** splash logo is still listed in ``m_SplashScreenLogos``. To fix this, we can simply delete the array item that contains the **Made with Unity** logo. Great!  
![Removing Made with Unity logo from Splash Screen Logos array](./img/IMG_PC_Mono_05.png "Removing Made with Unity logo from Splash Screen Logos array")
- Now let's move on to other settings. Move to **Asset List** and open ``BuildSettings``:  
<img id="img-enable-pro-version" src="./img/IMG_PC_Mono_06.png" alt="Viewing BuildSettings Unity 2019" title="Viewing BuildSettings Unity 2019" />

- Select ``hasPROVersion`` and change the value to ``true``:  
![Enabling Pro Version](./img/IMG_PC_Mono_07.png "Enabling Pro Version")
- Save your changes (IDK why **Apply** does nothing). UABE cannot overwrite the file being edited, so just save it as ``globalgamemanagers-mod``. Close ``globalgamemanagers`` on UABE or simply close UABE app, delete ``globalgamemanagers`` file (Remember! you have a backup, don't worry...) and rename ``globalgamemanagers-mod`` to ``globalgamemanagers``.  
![Save changes](./img/IMG_PC_Mono_08.png "Save changes")
- Try running the game and be surprised!

#### Fix for Unity >= 2020

If you've read all the steps above, you should know what the problem is, UABE can't deserialize PlayerSettings because it's encrypted (probably). That's why the Hex Editor is used for! Make sure you meet [prerequisite](#prerequisite)!

- Open ``globalgamemanagers`` file using HxD.
- Look for **Decoded Text** which contains **Company Name** and **Product Name** (in my case it's offset: 1060). Notice the **first Question Mark (?)** after the product name, right after that there's a hex value ``01 01`` (highlighted). If you change the hex value of the first ``01`` to ``00`` then the Splash Screen will be disabled, while the following ``01`` sets whether the **Made with Unity** logo will appear or not. If you don't want **Made with Unity** to appear, just change it to ``00``.  
![Find Splash Screen boolean](./img/IMG_PC_Mono_09.png "Find Splash Screen boolean")
- Boolean Splash Screen (``m_ShowUnitySplashScreen``) was discovered by the YouTube channel **Awesomegamergame** in his video <https://www.youtube.com/watch?v=xvh0AeZCX9E>. But the way he just change the Splash Screen boolean value to false doesn't work in other versions of Unity. I think this article as a complement to the tutorial.
- After changing the Splash Screen boolean, now change the ``m_hasPROVersion`` boolean exactly like [the method above](#img-enable-pro-version).

### Android (Mono, IL2CPP)

- Prepare a Unity game.  
![Prepare a Unity game](./img/IMG_Android_Mono_IL2CPP_01.png "Prepare a Unity game")
- Open APKToolGUI (Make sure you meet [APKToolGUI requirements](https://github.com/AndnixSH/APKToolGUI#requirements "APKToolGUI requirements")). Drag and Drop the APK file (which you guys think is the main APK, in my case ``base.apk``) to the APK Files section. APKToolGUI will automatically decompile the APK.  
![Decompile APK using APKToolGUI](./img/IMG_Android_Mono_IL2CPP_02.png "Decompile APK using APKToolGUI")
- Go to the decompiled APK folder and go to ``assets/bin/Data`` folder, you will find the ``globalgamemanagers`` file.  
![Find globalgamemanagers file](./img/IMG_Android_Mono_IL2CPP_03.png "Find globalgamemanagers file")
- **Do the same as in [PC (Mono) section](#pc-mono "PC (Mono) section")**.
- After finish editing ``globalgamemanagers``, go to APKToolGUI and re-compile the decompiled APK folder.  
![Recompile APK](./img/IMG_Android_Mono_IL2CPP_04.png "Recompile APK")
- DONE!
- For those who have uploaded the game to the App Store (such as Google Play), make sure to sign your APK consistently. If not, when you update your game that is already installed on Android, an error ```Signature is inconsistent with an existing application``` will appear. For new apps, it might be time for you to start signing APKs consistently. You can generate Keystore in Unity, go to ``Project Settings > Publishing Settings``.  
![Signing APK](./img/IMG_Android_Mono_IL2CPP_05.png "Signing APK")

## Notes

- There is no difference affecting the splash screen in the ``globalgamemanagers`` file between Development builds and Release builds.
- Yep, that's my Android game I used as a test subject. You can see find it at [Google Play Store](https://play.google.com/store/apps/details?id=aili.dev.meongadventure&hl=id&gl=US&pli=1 "Meong Adventure on Gogle Play Store") XD ( Pssttt... a remake is in the works).

## Huge Thanks

Many thanks to this fellas:

- [SeriousCache](https://github.com/SeriousCache "SeriousCache") for [Unity Asset Bundle Extractor (UABE)](https://github.com/SeriousCache/UABE "Unity Asset Bundle Extractor")
- Maël Hörz for [HxD Hex Editor](https://mh-nexus.de/en/hxd/ "HxD Hex Editor")
- [iBotPeaches](https://github.com/ibotpeaches "iBotPeaches") for [Apktool CLI](https://ibotpeaches.github.io/Apktool/ "Apktool CLI")
- [INF1NUM](https://github.com/INF1NUM "INF1NUM") | [AndnixSH](https://github.com/AndnixSH "AndnixSH") for [APKToolGUI](https://github.com/AndnixSH/APKToolGUI "APKToolGUI")
- [Awesomegamergame](https://www.youtube.com/@Awesomegamergame "Awesomegamergame YouTube channel") for [discovering splash screen offset](https://www.youtube.com/watch?v=xvh0AeZCX9E)
- and others that I can't mention...

## Disclaimer

By doing this, of course you violate the applicable terms of Unity. #DWYOR!
