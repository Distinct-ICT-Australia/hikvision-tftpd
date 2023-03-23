# Introduction

Factory reset a Hikvision NVR or Camera over the a network.
Useful for when:

1. NVR has bricked itself when doing a firmware upgrade. Or for whatever else reason...
2. Cameras are unavailable due to being locked by a password.

## Support

The below tables shows when the tool was last tested.

| Windows    | MacOS      | Linux      |
| ---------- | ---------- | ---------- |
| 2023-03-22 | 2023-03-22 | 2023-03-02 |

If you require further assistant please reach out to [support@distinctict.com.au](mailto:support@distinctict.com.au).

## Script Setup Steps

1. Download Python 2.7 
2. Clone this repository.
3. Collect your digicap firmware version: `curl -o digicap.dav <url_of_firmware>`.
4. Ensure the digicap file is in your working directory.

## System Configuration (SERVER, not NVR or Camera)

### Context

The client (NVR or Camera) sends a particular packet to the server's port 9978 from the clients port 9979 and expects the server to echo it back. Once that happens, it proceeds to send a TFTP request over port 69 for a specific file, which the client will then install. The TFTP Server must reply from port 69.

This script handles both the handshake and the actual TFTP transfer.

Note the expected IP Addresses and file name are different based on the model. Currently there are two known configurations, please submit a PR if you identify additional.

| client IP    | server IP    | filename      |
| ------------ | ------------ | ------------- |
| 192.0.0.64   | 192.0.0.128  | `digicap.dav` |
| 172.9.18.100 | 172.9.18.80  | `digicap.mav` |

The script defaults to 192.0.0.128, below we define how to change to a different Server IP.

### Ethernet Adapter

1. Connect your device directly to the client via ethernet.
2. Adjust your ethernet adapters settings to reflect the following. If your client IP reflect a `172` address, change the IPv4 field to `172.9.18.80`.
	| IPv4        | Subnet Mask   | Gateway         |
	| ----------- | ------------- | --------------- |
	| 192.0.0.128 | 255.255.255.0 | 0.0.0.0 (blank) |

## Execution process (Default)

1. Launch the server:
- Windows: `python hikvision_tftp.py`.
- MacOS & Linux: `sudo ./hikvision_tftpd.py`.
2. Restart the client.
3. Once the client reboots, you should see a transfer from the server start to print into the servers console. 
4. Once the transfer is complete you can close the script. 

## Execution process (Other server IPs)

1. Launch the server
- Windows: `python hikvision_tftp.py --server-ip=172.9.18.88 --filename=digicap.mav`
2. Restart the client.
3. Once the client reboots, you should see a transfer from the server start to print into the servers console. 
4. Once the transfer is complete you can close the script. 

Now leave the device to perform a factory reset. It may reboot a number of times, do not interfere with the process until it is clearly complete.

If nothing happens when your device restarts, your device may be expecting
another IP address. tcpdump may be helpful in diagnosing this:

    $ sudo tcpdump -i eth0 -vv -e -nn ether proto 0x0806
    tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
    16:21:58.804425 28:57:be:8a:aa:53 > ff:ff:ff:ff:ff:ff, ethertype ARP (0x0806), length 60: Ethernet (len 6), IPv4 (len 4), Request who-has 172.9.18.80 tell 172.9.18.100, length 46
    16:22:00.805251 28:57:be:8a:aa:53 > ff:ff:ff:ff:ff:ff, ethertype ARP (0x0806), length 60: Ethernet (len 6), IPv4 (len 4), Request who-has 172.9.18.80 tell 172.9.18.100, length 46

Feel free to open an issue for help.

