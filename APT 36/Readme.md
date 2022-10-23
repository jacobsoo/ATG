# APT 36

On 26th August 2020, Kaspersky listed the Android samples of APT 36  on their [blog post](https://securelist.com/transparent-tribe-part-2/98233/). The samples seems simple and non sophisticated.

A rather [newer sample](https://www.virustotal.com/gui/file/fafcbb35db7cd2725d2f3f4268ffb32390f0e7602263841914fae72f37baca5b/details) were uploaded on Virustotal on 23rd April 2021.
The C2 found in that sample is `109[.]236[.]85[.]16:5987` / `myabcxyz1[.]ddns[.]net:5987`

If you have your own custom made Yara for Android, you should create a rule that checks if the APK contains the following
 - `.MainS` in Services
 - `.MyReceive` & `.CallReceive` in Receivers
 
 This is a relatively good method for detecting future samples if they continue to use it.
