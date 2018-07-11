Requests

+ A required "method" string telling the name of the method to invoke
+ An optional "arguments" object of key/value pairs
+ An optional "tag" number used by clients to track responses.



Responses

+ A required "result" string whose value MUST be "success" on success,
         or an error string on failure.
+ An optional "arguments" object of key/value pairs
+ An optional "tag" number as described in 2.1.







#### Torrent Action Requests

 Method name          | libtransmission function
   ---------------------+-------------------------------------------------
   "torrent-start"      | tr_torrentStart
   "torrent-start-now"  | tr_torrentStartNow
   "torrent-stop"       | tr_torrentStop
   "torrent-verify"     | tr_torrentVerify
   "torrent-reannounce" | tr_torrentManualUpdate ("ask tracker for more peers")

#### Torrent Mutators

Method name: "torrent-set"

#### Torrent Accessors

Method name: "torrent-get".

#### Adding a Torrent

 Method name: "torrent-add"

```
{
  "method": "torrent-add",
  "arguments": {
    "filename":"/Users/liuwei/Downloads/ubuntu-16.04.4-desktop-amd64.iso.torrent",
    "download-dir":"/Users/liuwei/Downloads/"
  }
}
```



```
{
    "arguments": {
        "torrent-added": {
            "hashString": "778ce280b595e57780ff083f2eb6f897dfa4a4ee",
            "id": 1,
            "name": "ubuntu-16.04.4-desktop-amd64.iso"
        }
    },
    "result": "success"
}
```



#### Removing a Torrent

Method name: "torrent-remove"

#### Moving a Torrent

Method name: "torrent-set-location"

#### Renaming a Torrent's Path

Method name: "torrent-rename-path"

### Session Requests???