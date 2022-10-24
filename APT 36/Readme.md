# APT 36

On 26th August 2020, Kaspersky listed the Android samples of APT 36  on their [blog post](https://securelist.com/transparent-tribe-part-2/98233/). The samples seems simple and non sophisticated.

A [newer sample](https://www.virustotal.com/gui/file/fafcbb35db7cd2725d2f3f4268ffb32390f0e7602263841914fae72f37baca5b/details) were uploaded on Virustotal on 23rd April 2021.

The C2 found in that sample is `109[.]236[.]85[.]16:5987` / `myabcxyz1[.]ddns[.]net:5987`

If you have your own custom made Yara for Android, you should create a rule that checks if the APK contains the following
 - `.MainS` in Services
 - `.MyReceive` & `.CallReceive` in Receivers
 
 This is a relatively good method for detecting future samples if they continue to use it.
 The folowing are the hashes which we can find from VT base on the above clues.
 
 ## Hashes
 - `0c5b37b48769df1f88d84137c2084bd023b7d6d44a3bdc62ef8c370f3c15fec5`
 - `52d1cb75b7826e57b332ba56b5395d2666dfc0cd363b3f0910c909ad74e67f8f`
 - `58b63b21d2ac45e5bc2824101512804831f5a618b26ec132496c292f55693c1e`
 - `265f487a2972b8ea5fea32e7c7ac7546d12dff56374ea6dcc3ade0266c4c19cb`
 - `469b51a10f9bb41c2606fccc2109d80918dd83e619f801d3e5c0b772060b7e4b`
 - `717d42ca72868c9dd55a36e04af44285e9dbf5f679b8abfbc2c213f5522cf726`
 - `720ecdd4e04883399763c38e3de4e5ec525b1879d44d926b906e38d5c3edbad2`
 - `b31c0e284d5bddbfb1a5924a8c2d6fda335fc0e4a25560afcfe9c9b11ecb68bc`
 - `c63b6073e54d433c009312f12dd6be482ce3bbb5b49d536ee013b7ccbc1a4d3e`
 - `1d806466896998a6c4ac962d6e5381fe704670e3fd912db98e13c2a5482b9a7e`
 - `2e103dd8eda4750fd5fe99c0c5fbc987ae7712bea7c08db4db240e85a0ee1bbf`
 - `2edbf7f0f563794a12ba8779b9f3ddc2677e49ae5abd9a43e46879b828f50cd2`
 - `3bff8e9d7a1bdd3ac99c1ed9c00bfaf62cfdbb1d078f85977917b82cfe7045cc`
 - `5a38b73883ad2247d4599ea191f9f39d527b16bdcad787ae443a03e9116d02b5`
 - `5ce6d273df4fcad8953dd4f37e7f4a0390f9da52978f0227b021b6a724b59313`
 - `7a4150b94f130a4f41110315e8f1df14269ed3559aede6ee5228008767a59af4`
 - `9e04a80368ec9ac90d99c36a12c64bd0437d9e5cf5d13f0e802126cf4ead8f9f`
 - `56adffc9852e802d43edacb1202e7681577ddfbee9d63cecabc0d50e29e38840`
 - `56bba40bc62d021b17fb79d1e7e307a61a8727bf9f59fdcae8cfff7a0e0f9422`
 - `74f568344d7e29cc3d184de533864dfa5e085f8090ec1a88432c1d7f12c444bc`
 - `81ca347465f28d093a27caf3d83fe5c4fb50c5e48cfd851a06784d431fa8c2cf`
 - `451cab3d5b5a1c699f1b9b3d8a4ddf73c48f891cbca88e4e1a829e2442116efb`
 - `464f197009b2a2c3c8d52cf97c9615871c9f83f30858a771870fc9b836e0e4d4`
 - `639e5902254d039690f8132cdbaa2acd40acfda77f5a40398da7b04b2500df38`
 - `3565e0f2205a1aa6c3e095b6f0a8621dbc2ae6d6c84efc1e9d5a766c288ed6f3`
 - `989002aae45f54021330ddb8cc1c9fcefe2f862a55304957b222ad1824839593`
 - `58643719af0f271b87c51665c2e8c904db70155b8c6f514d6e5f44c0828a4a53`
 - `c598f7956b1d0d6514ade39df05c9fcad70f970957c980d15f6d019e5ca04df6`
 - `ccbe720fd059610227d478578a5a4019c96885de8fd3e83984f9c1c5fe850ad6`
 - `cee88ffae63cd95e4f9f7008a86a8d7818a47b62608d28a70bbe8d73c6d2b042`
 - `e158b20c400cb2c24ce4223bb947dee86e3717b18446b76911389a9a1fcc2260`
 - `fadc0fe0714d9e95904ccaaf16d895ac71c1337e92b8eb51258fed9fb8fb4620`
 
## Dropper
The following are the droppers.
- `53e3d197e9d6ac3f23f8da64ffb1a1013e84d1ebf52864f5b0f8b79006f1ddc4`
- `72aa69be5cd46220e1509c040ceb6e3cbb3c676a6c464a811370d688f45f26ec`
