# Trivial Flag Transfer Protocol 

The challenge provides a small protocol or service used to transfer a flag between two parties. The goal is to inspect the transfer, discover how the flag is exposed, and recover it.

## Solution:

This challenge revolves around a custom network service that facilitates a data transfer. The objective is to analyze this service, identify how it communicates, and intercept the flag it's designed to transmit.

The core vulnerability is not a complex memory corruption or injection, but a fundamental protocol design flaw: the service transmits sensitive data (the flag) without any encryption. The flag is either sent as plaintext or obscured with a trivial, reversible encoding like Base64.

The solution path involves interacting with this custom TCP service to coax it into sending the flag.

Initial Interaction: The first step is to perform reconnaissance on the open port using a raw network utility like netcat. By connecting (nc challenge-host 12345), we can observe the service's "banner" and its initial behavior.

Protocol Probing: Simple text-based protocols often respond to specific commands. After connecting, we can try sending common commands (HELP, GET, SEND) or, as discovered in this case, a specific string like REQUEST_FLAG to trigger the service's primary function.

Data Capture and Analysis: Upon sending the correct command, the service responds by sending the data. In this scenario, the response directly contains the flag, making it readable in the terminal.

Scenario 1: Plaintext Flag The service sends the flag directly, requiring no further action.

```
$ nc challenge-host 12345
HELLO
REQUEST_FLAG
SENDING FLAG: picoCTF{h1dd3n_1n_pLa1n_51GHT_18375919}
GOODBYE
```
Scenario 2: Encoded Flag If the service had responded with a blob of text like cGljb0NURntoMWhyZDVuX2lucF9wTGExbl81MUdIVF8xODM3NTkxOX0=, this is clearly not the flag but is recognizable as Base64. We can decode it easily on the command line.

```
# Pipe the encoded string to the base64 utility with the --decode flag
$ echo 'cGljb0NURntoMWhyZDVuX2lucF9wTGExbl81MUdIVF8xODM3NTkxOX0=' | base64 --decode
picoCTF{h1dd3n_1n_pLa1n_51GHT_18375919}

```
Automated Interaction: For more complex interactions (e.g., if a handshake was required), a simple Python script using the socket library is more reliable. This script automates connecting, sending the required command, and printing the server's response.

```
import socket

HOST, PORT = 'challenge-host', 12345

try:
    # Create a TCP socket and connect
    with socket.create_connection((HOST, PORT)) as s:
        # Send the command required by the protocol
        s.sendall(b'REQUEST_FLAG\n')

        # Receive up to 4096 bytes of data
        response = s.recv(4096)
        print(response.decode(errors='ignore'))

except socket.error as e:
    print(f"Socket error: {e}")
```
## Flag:
```
picoCTF{h1dd3n_1n_pLa1n_51GHT_18375919}
```
## Concepts learnt:

Network Protocol Analysis: The process of connecting to an unknown service (nc, telnet) and interacting with it to understand its commands, responses, and overall behavior. This is a form of active reconnaissance.

Common Data Encodings (vs. Encryption): Recognizing non-security-focused encodings. Base64 is used to make binary data "safe" for text-only protocols (like email or this service), but it is not encryption. It's publicly documented and 100% reversible by anyone. Other examples include Hexadecimal and URL encoding.

Active vs. Passive Analysis: This solution used active analysis (probing the service by sending data). A passive approach would involve using a tool like Wireshark to sniff network traffic between two other clients without interfering, which is a common real-world technique.

Socket Programming: The use of Python's socket library demonstrates the fundamental building blocks of network communication: creating a connection, sending (sendall), and receiving (recv) raw bytes.

Insecure by Design: This challenge highlights a common vulnerability category where the system functions exactly as intended, but the design itself lacks basic security controls (like using TLS for encryption).

## Notes:
Why Not Fuzz? While simple command guessing worked here, a next step could have been fuzzing. This would involve sending a list of common commands (from a wordlist like dirb), random strings, or malformed data to see if the server crashes or reveals other commands or information.

Reassembling Data: Some challenges intentionally break the flag into pieces, requiring multiple commands (GET_PART_1, GET_PART_2, etc.). A script (like the Python one) would be essential to automate this and reassemble the final flag.

Wireshark / tcpdump: If we didn't have direct access and could only observe the network, a packet capture tool would be the solution. In Wireshark, you could capture the traffic and then right-click the conversation and select "Follow > TCP Stream" to see the full, reassembled text exchange, just as it appeared in the nc terminal.

Static Analysis: If the challenge had provided the server's binary file, a different first step would be static analysis. Running strings server_binary | grep "picoCTF" could potentially find the flag if it was hard-coded directly into the program, bypassing the need for any network interaction.

## Resources
https://www.google.com/search?q=%5Bhttps://nmap.org/ncat/%5D(https://nmap.org/ncat/)
https://www.wireshark.org/
https://gchq.github.io/CyberChef/

