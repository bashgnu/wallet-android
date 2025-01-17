Beta channel
============

In order to receive updates quicker than others, you need to enable beta versions of the software in
[Google Play](https://play.google.com/apps/testing/com.mycelium.wallet)

As beta testers, please make sure you have a recent **backup of the masterseed** and all **private keys** inside Mycelium. Beta testers will experience many bugs.
So far, restoring the wallet from masterseed has never been necessary, but we offer no guarantees.

Building
========

To build everything from source, simply checkout the source and build using gradle on the build system you need:

 * JDK 1.7

Then you need to use the Android SDK manager to install the following components:

 * `ANDROID_HOME` environment variable pointing to the directory where the SDK is installed
 * Android SDK Tools 22.6.4
 * SDK Platform Tools 19.0.2
 * Android SDK build Tools 19.1.0
 * Android 4.4.2 (API 19) (at least SDK Platform Rev. 3)
 * Android Extras:
    * Android Support Repository rev 5
    * Android Support Library rev 19.1
    * Google Play services for Froyo rev 12
    * Google Play services rev 17
    * Google Repository rev 8


The project layout is designed to be used with a recent version of Android Studio (currently 1.1.0)

#### Build commands

To get the source code, type:

    git clone https://github.com/mycelium-com/wallet-android.git
    cd wallet-android
    git clone https://github.com/mycelium-com/wallet-android-modularization-tools.git
    git submodule update --init --recursive

Linux/Mac type:

    ./gradlew clean test mbw::asProdRel mbw::asBtRel

Windows type:

    gradlew.bat clean test mbw::asProdRel mbw::asBtRel

 - Voila, look into `mbw/build/outputs/apk/` to see the generated apk.
   There are versions for both prodnet and testnet.

Alternatively you can install the latest version from the [Play Store](https://play.google.com/store/apps/details?id=com.mycelium.wallet).

If you cannot access the Play store, you can obtain the apk directly from the Mycelium Bitcoin
Wallet [download page](https://mycelium-wallet.s3.amazonaws.com/apks/mbw-prodnet-release.apk).

Deterministic builds
====================

To validate the Mycelium image you obtain from Google Play Store, you can rebuild the Mycelium
wallet yourself using Docker and compare both images following these steps:

* Get the source as above
* Create your own Docker image from our simple Dockerfile

        $ docker build . --tag mycelium-wallet

  Check that this step succeeds by listing the available docker images:

        $ docker images | grep mycelium-wallet

* Use disorderfs to eliminate non-determinism caused by file ordering

        $ mkdir /tmp/s
        $ sudo disorderfs --sort-dirents=yes --reverse-dirents=no --multi-user=yes $PWD /tmp/s
        $ cd /tmp/s

* Build Mycelium using Docker

        $ mv local.properties local.properties.bak
        $ docker run --rm --volume $(pwd):/project --workdir /project -it mycelium-wallet bash
        # yes | /opt/android-sdk/tools/bin/sdkmanager "build-tools;28.0.3"
        # ./gradlew clean :mbw:assProdRel
            
  `local.properties` can break docker compiles. Move it out of the way. Run the
  mycelium-wallet docker with gradle compilation of mbw.
  As docker
  might run as a different user, its generated files will also be "not yours". Make them yours using
  `chown` as super user.
  
  The app can now be found in `mbw/build/outputs/apk/prodnet/release/mbw-prodnet-release.apk`.
  
  As maintainer with release keys you want to run a slightly different command:
  Add these docker parameters: `--volume 'path/to/keys.properties':/project/keys.properties --volume 'path/to/keystore_mbwProd':/project/keystore_mbwProd --volume 'path/to/keystore_mbwTest':/project/keystore_mbwTest`
  Build all these targets `:mbw:assBtctRel :mbw:assProdRel :mbw:assBtctDeb :mbw:assProdDeb`
  and to turn the missing release keys into an error, add this gradle option `-PenforceReleaseSigning`
  
  After this step succeeds, all apks are in `/tmp/release_mbw/`.
  
  Note: for those who use Docker Toolbox $(pwd) should be under your home user folder since this is the [only folder that is shared with VM](https://github.com/docker/kitematic/issues/2738).

* Retrieve Google Play Mycelium APK from your phone
  Gets package path:

        $ adb shell pm path com.mycelium.wallet
        package:/data/app/com.mycelium.wallet-1/base.apk

  Retrieve file:

        $ adb pull /data/app/com.mycelium.wallet-1/base.apk mycelium-signed.apk
        
* Extract content from both apks you want to compare, using [ApkTool](https://ibotpeaches.github.io/Apktool/):

        java -jar ~/path/to/apktool.jar d mbw-prodnet-release.apk
        java -jar ~/path/to/apktool.jar d mycelium-signed.apk

* Compare signed apk with unsigned locally built apk using a diff tool such as meld

        diff --brief --recursive  mbw-prodnet-release/ mycelium-signed/ | grep -v "META-INF/CERT.RSA\|META-INF/CERT.SF\|META-INF/MANIFEST.MF"

* The expected difference between these files are elements that depend on the signature, that only
  the project's maintainer can reproduce:
  
  * `original/META-INF/CERT.RSA` 
  * `original/META-INF/CERT.SF` 
  * `original/META-INF/MANIFEST.MF` 

Features
========

With the Mycelium Bitcoin Wallet you can send and receive Bitcoins using your mobile phone.

 - HD enabled - manage multiple accounts and never reuse addresses ([Bip32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)/[Bip44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) compatible)
 - Masterseed based - make one backup and be safe for ever. ([Bip39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki))
 - 100% control over your private keys, they never leave your device unless you export them
 - No block chain download - install and run in seconds
 - Ultra fast connection to the Bitcoin network through our super nodes
 - For enhanced privacy and availability you can connect to our super nodes via a tor-hidden service ( *.onion* address)
 - Watch-only addresses (single or xPub) & private key (single or xPriv) import for secure cold-storage integration
 - Directly spend from paper wallets (single key, xPriv or master seed)
 - Trezor enabled - directly spend from trezor-secured accounts.
 - [Mycelium Entropy](https://mycelium.com/entropy) compatible Shamir-Secret-Shared 2-out-of-3 keys spending
 - Secure your wallet with a PIN
 - Compatible with other bitcoin services through the `bitcoin:` URI scheme


Please note that bitcoin is still experimental and this app comes with no warranty - while we make sure to adhere to the highest standards of software craftsmanship we can not exclude that the software contains bugs. Please make sure you have backups of your private keys and do not use this for more than you are willing to lose.

This application's source is published at https://github.com/mycelium-com/wallet
We need your feedback. If you have a suggestion or a bug to report [create an issue](https://github.com/mycelium-com/wallet/issues).

More features:
 - Sources [available for review](https://github.com/mycelium-com/wallet-android)
 - Multiple HD accounts, private keys, external xPub or xPriv accounts
 - Multiple Bitcoin denominations: BTC, mBTC, bits and uBTC
 - View your balance in multiple fiat currencies: USD, AUD, CAD, CHF, CNY, DKK, EUR, GBP, HKD, JPY, NZD, PLN, RUB, SEK, SGD, THB, and many more
 - Send and receive by specifying an amount in fiat and switch between fiat and BTC while entering the amount
 - Address book for commonly used addresses
 - Transaction history with detailed information and local stored comments
 - Import private keys using SIPA (the ones beginning with a 5) and mini private key format (Casascius private keys) from QR-codes or clipboard
 - Export private-, xPub- or xPriv-keys as QR-codes, on clipboard or share with other applications
 - Share your bitcoin address using Twitter, Facebook, email and more.
 - Integrated QR-code scanner
 - Client side load balancing between three 100% redundant server nodes located in different data centers.
 - Sign Messages using your private keys (compatible with bitcoin-qt)

Authors
=======
 - Jan Møller
 - [Andreas Petersson](https://github.com/apetersson)
 - [Daniel Weigl](https://github.com/DanielWeigl)
 - [Jan Dreske](https://github.com/jandreske)
 - Dmitry Murashchik
 - Constantin Vennekel
 - [Leo Wandersleb](https://github.com/Giszmo)
 - [Daniel Krawisz](https://github.com/DanielKrawisz)
 - [Jerome Rousselot](https://github.com/jeromerousselot)
 - [Nelson Melina](https://github.com/DaLN)
 - [Elvis Kurtnebiev](https://github.com/xElvis89x)
 - [Sergey Dolgopolov](https://github.com/itserg)
 - [Sergey Lappo](https://github.com/sergeylappo)
 - Alexander Makarov
 - [Nadia Poletova](https://github.com/poletova-n)
 - [Kristina Tezieva](https://github.com/agneslovelace)
 - [Nuru Nabiyev](https://github.com/NuruNabiyev)
 

Credits
=======
Thanks to all collaborators who provided us with code or helped us with integrations!
Just to name a few:

 - [Nicolas Bacca from Ledger](https://github.com/btchip)
 - Sipa, Marek and others from Trezor
 - Jani and Aleš from Cashila
 - [Kalle Rosenbaum, Bip120/121](https://github.com/kallerosenbaum)
 - David and Alex from Glidera
 - [Wiz](https://twitter.com/wiz) for helping us with KeepKey
 - Tom Bitton and Asa Zaidman from Simplex
 - (if you think you should be mentioned here, just notify us)

Thanks to Jethro for tirelessly testing the app during beta development.

Thanks to our numerous volunteer translators who provide high-quality translations in many languages. Your name should be listed here, please contact me so I know you want to be included.

Thanks to Johannes Zweng for his testing and providing pull requests for fixes.

Thanks to all beta testers to provide early feedback.
