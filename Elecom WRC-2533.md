# Elecom WRC-2533GST Router

Last weekend, I spent some time exploring the firmware of my router - the Elecom WRC-2533GST. Initially, I was just curious and doing some basic analysis. However, after downloading its firmware and digging deeper, I was shocked by the number of vulnerabilities I discovered.

The Elecom WRC-2533GST is an older device that has reached its end of support. Despite this, it could still remain widely used—probably because most people are quick to upgrade their phones but rarely think about the router tucked away in some corner of their home. Let’s be honest: who wants to mess with the "mystery box" that just works?

## Vulnerabilities in Command Execution

One of the alarming issues I found was the frequent use of `system` and `popen` calls in the firmware to process user requests. These calls, if not properly sanitized, can easily lead to command injection vulnerabilities.

![System Calls](images/wrc/systemxref.png?raw=true "System Calls")

## Hidden Features and Command Injection

Even more surprising was the presence of features in the firmware that don’t actually exist in the router’s UI. For instance, the `usbapps` feature, which is designed to manage storage, can still process requests if the input data is structured correctly.

![usbapps Function](images/wrc/usbappsfunction.png?raw=true "Code process usbapps function")

Don't be fool by the `uci_safe_get` function call to get the data out of config, in reality, "safe" here does not guarantee proper input sanitization. This allows attackers to exploit configuration values such as `usbapps.config.smb_admin_name` and `usbapps.@smb[%d].username` to execute arbitrary commands on the router.

![HTTP Request to usbapps](images/wrc/request.png?raw=true "HTTP request to usbapps")

## The Fix in the Successor Model

Elecom has addressed command injection issues in their newer products. For example, in the WRC-2533GST2 (the successor to the WRC-2533GST), the firmware includes checks on POST data to filter out potentially dangerous characters `"&;|><\"``\*\\$"`.

![The Fix](images/wrc/filter.png?raw=true "The fix")

Does filtering all these characters prevent command injection exploit payloads in POST data? It works, I guess. If you know of any ways to bypass this filter, feel free to share—I’d love to learn more!

Unfortunately, even with these protections, I was still able to gain a shell on the WRC-2533GST2 (of course by command injection).

![Root Access](images/wrc/WRC-2533GST2root.png?raw=true "Still be rooted")

If anyone has contact information for Elecom’s security team, please let me know. I would like to report this issue along with several other vulnerabilities.

## Upgrading the WRC-2533GST

As for the original WRC-2533GST, it’s unfortunate to see such a reliable device become vulnerable over time. However, there is a potential solution: you can upgrade it using the firmware from the WRC-2533GST2. If you have time, I recommend giving it a try.

