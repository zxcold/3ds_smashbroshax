This is haxx for Super Smash Bros for 3DS, via local-WLAN beacon haxx. The haxx triggers while the application is scanning for local multiplayer sessions, when the beacon is being broadcasted.
In certain cases the application may somewhat hang or crash prior to any actual ROP being run: this hax is not completely reliable, one reason is because the heap layout not always being in the intended state. Another reason(which actually seems to be the main cause usually) is that the ROP may fail to recv another beacon quickly enough, which results in jumping to using a stack which is all-zeros(there's no good way to do error checking/looping for that, partly because of lack of space). Also note that in some cases it may take a while for the hax to trigger.

Since this is all contained in a single wifi beacon, the amount of space available for the hax is very small: currently almost all of this space is used.

This .git was originally created on September 26, 2014.

This will not work on 3DS systems where config-mem UNITINFO(http://3dbrew.org/wiki/Configuration_Memory) is clear for dev-unit, unless you manually adjust the COMMID values in the Makefile.

Note that because this is a local-WLAN beacon broadcast, *all* 3DS systems in range doing regular smash-3ds multiplayer session scanning will be affected by doing this broadcasting: either the system would crash/etc(such as when the hax version doesn't match the app version), or code would run on the system(which normally would only end up executing an infinite loop due to failing to load the payload, unless the hax was built with PAYLOADURL). Therefore, please don't broadcast this when there's 3DS systems in range which are not your own doing the above scanning.

The Smash 3DS code handling beacons from Wii U does not involve the vulnerable function used with the normal multiplayer beacon handling.

# Versions
Supported application builds:
* demo: USA+EUR supported and tested. There's no difference between the regular demo and the "Special Demo" with this hax. This was the only version of Smash-3ds supported by this hax initially, until after the USA version of the game was released.
* v1.0.0. USA: supported+tested. "gameother": supported+tested.
* v1.0.2. USA: supported, not tested.
* v1.0.4. USA: supported+tested. "gameother": supported, not tested.
* v1.0.5. USA: "supported". The target heap address for overwriting the target object varies, hence this hax doesn't actually work right with this version. This version is not fully supported due to this.
* v1.1.0. USA: supported+tested. "gameother": supported+tested.

Last version tested with this vuln was v1.1.0, vuln still isn't fixed with that version.

EUR and JPN full-game .code binaries addresses-wise are basically the same, for v1.0.4 at least. Hence, the filenames for these two regions include "gameother".

This can't be completely blocked with the main app without a system-update: even if an app-update would fix it, one could just rename/whatever the update-title directory on SD card to force the system to not use the update-title(the directory name could be restored to the original later when not using this hax). One could also do this if the currently installed update-title version is not supported, or when the latest version of the update-title isn't supported(where the currently installed version isn't supported).  
The above mentioned directory is at the following SD card location: "/Nintendo 3DS/{ID0}/{ID1}/title/0004000e/{TIDHigh}".
Where TIDHigh for the update-title is one of the following:
* USA: 000edf00
* EUR: 000ee000
* JPN: 000b8b00

# Building
ctr-wlanbeacontool from here is required: https://github.com/yellows8/ctr-wlanbeacontool

Make params:
* "INPCAP={path}" This is the smash-beacon to use as a base. When used, the specified beacon is used instead of the default one("smashdemo_beacon_modbase.pcap").
* "PAYLOADURL={url}" HTTP URL to download the payload from.
* "PAYLOADPATH={sdpath}" SD path to load the payload from, for example: "PAYLOADPATH=/smashpayload.bin".
* "ADDITONALDATA_SIZE1={val}" Use the specified value for the raw size value in the beacon additionaldata, instead of the default one. For example, with a large value this could be used to build vuln-test pcaps.
* "BEACON_BYTEID={val}" Override the u8 ID used in the networkstruct(default is 0x0 for regular Smash multiplayer). 0x1 is smash-run(which works fine with this hax). 0x55 is WiiU comms: the vulnerable smash-3ds function is never executed with this however.

Only one of the PAYLOAD* params must be specified.

# Usage
Remember to always broadcast the beacon on the same channel as specified in the beacon itself(channel 6 with the default pcap base). The following data *must* *not* be changed in the beacon frame while it's being sent: host/BSSID MAC addresses, and all of the beacon tags. The MAC address with the default base pcap is: 59:ee:3f:2a:37:e0.

The built beacon-hax pcaps are located under "pcap_out/". In the filenames, "vXYZ" means game-version "vX.Y.Z". Full-game filenames for USA include "gameusa", while the other regions filenames include "gameother".

One way to send the beacon is with aireplay-ng, however that requires a patch, see aireplay-ng.patch. For example, to send the beacon with aireplay-ng(the wifi interface must already be in monitor mode + be on the correct channel): aireplay-ng --interactive -r {beaconpcap_path} -h {host mac from pcap} -x 10 {wifi interface}

This can be used with the homebrew-launcher otherapp payload to boot into hbmenu. However, doing so is New3DS-only, at the time of writing(supporting Old3DS with hb-launcher from this would require major changes). With game-version v1.1.0, the initial otherapp payload gfx isn't displayed correctly(when home-menu takeover is being done), however it's fine after that(this would ideally be fixed via an updated hb-launcher payload).

Due to the different FCRAM memregions layout on Old3DS with smashbros, the system will do a hardware reboot when the smashbros process terminates. This happens when exiting the title via Home Menu. This also happens automatically when attempting to boot the hb-launcher payload on Old3DS. Therefore, the hb-launcher payload can only be booted successfully on New Nintendo 3DS and New Nintendo 3DS XL(at the time of writing).

Right after the initial arm11code initializes stack, it will overwrite the framebuffers in VRAM with junk, to indicate that the code is running. Originally this was intended for the top-screen, however with v1.1.0 on new3ds this ends up only overwritting the bottom-screen framebuffers.

The baseaddr for the payload is 0x00111000, max size is 0xa000-bytes. Whenever loading the payload fails, the arm11code will just execute an infinite loop.

# Homebrew Launcher Payload
With the release builds, the hax loads the payload from SD "/smashpayload.bin". This should contain the hb-launcher(https://smealum.github.io/3ds/) otherapp payload. Until there's a proper otherapp payload selector on the hb-launcher site, the payload can be downloaded from the following URL(see also https://github.com/smealum/sploit_installer):  
* "https://smealum.github.io/ninjhax2/Pvl9iD2Im5/otherapp/{PAYLOADNAME}.bin"
Where {PAYLOADNAME} is: "FIRMVER_REGION_MENUVER_MSETVER".  

FIRMVER values(without quotes):
* "POST5" = non-New3DS
* "N3DS" = New3DS

REGION values(without quotes):
* "U" = USA
* "E" = EUR
* "J" = JPN

MENUVER values(without quotes):
* "11272": Non-JPN, system-version v9.0.
* "12288": System-version v9.2.
* "13330": System-version v9.3.
* "14336": System-version v9.4.
* "15360": System-version v9.5.
* "16404": System-version v9.6.
* "17415": System-version v9.7.
* "20480_usa": USA, system-versions v9.9-v10.0.
* "19456": Non-USA, system-versions v9.8-v10.0.

MSETVER values(without quotes):
* "8203": System-versions below v9.6.
* "9221": System-versions starting with v9.6.

For example, the payload URL for New3DS USA 9.9.0-X - 10.0.0-X is: https://smealum.github.io/ninjhax2/Pvl9iD2Im5/otherapp/N3DS_U_20480_usa_9221.bin

# Usage Guide
* 1) Download the release archive.
* 2) Download the otherapp payload to SD "/smashpayload.bin", as described under the "Homebrew Launcher Payload" section.
* 3) Determine which pcap in the "pcap_out" directory to use, see the "Usage" section.
* 4) In wifi monitor mode, begin broadcasting the pcap on channel 6 with MAC address "59:ee:3f:2a:37:e0", with the beacon being sent at least 10 times per second.
* 5) On the target 3DS system, start Super Smash Bros, then goto "Smash" -> "Group".
* 6) The hax will then eventually trigger on the 3DS system.

