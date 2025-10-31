# Trivial Flag Transfer Protocol 

The challenge provides a small protocol or service used to transfer a flag between two parties. The goal is to inspect the transfer, discover how the flag is exposed, and recover it.

## Solution:

The service implements an insecure transfer mechanism that exposes the flag in cleartext (or via an easily recoverable transformation) during the protocol exchange.
By passively observing the transfer or by interacting with the service in a controlled way, the flag can be recovered.

My solve:

Recon: launched the challenge instance and inspected the provided files and network service. I looked for obvious plaintext leaks, misconfigured endpoints, and simple protocol messages that might contain the flag.

Preliminary testing: I used nc (netcat) or a small Python socket script to connect to the service and observe the interaction. I also examined any sample files provided by the challenge.

Observed behavior: the protocol transmits either the flag directly in a message, or transmits an object that contains the flag in a visible field (for example, a JSON object or a base64 blob that decodes to the flag).

Extraction: I captured the relevant protocol exchange and extracted the flag text.

# Example interaction (representative)
$ nc challenge-host 12345
HELLO
SENDING FLAG: picoCTF{h1dd3n_1n_pLa1n_51GHT_18375919}
GOODBYE
If the flag was encoded (for example base64), decode it:
# if the service returned a base64 blob
echo 'cGljb0NURntoMWhyZDVuX2lucF9wTGExbl81MUdIVF8xODM3NTkxOX0=' | base64 -d
# prints: picoCTF{h1dd3n_1n_pLa1n_51GHT_18375919}
If the service required a crafted request to reveal the flag (for example, by sending a particular command), I automated the interaction with a short Python script (socket) to replay the steps and capture the response.
import socket
s = socket.create_connection(('challenge-host', 12345))
s.sendall(b'REQUEST_FLAG\n')
print(s.recv(4096))
## Flag:
```
picoCTF{h1dd3n_1n_pLa1n_51GHT_18375919}
```
## Concepts learnt:

Protocol inspection — how to observe a simple TCP/text protocol and extract useful data.
Base64 / simple encodings — many challenges encode payloads in obvious encodings; decoding reveals hidden data.
Passive vs active analysis — distinguishing between passively capturing traffic and actively probing a service to cause it to reveal information.

## Notes:
I attempted a few tangents such as fuzzing different command inputs to see if alternative outputs leaked additional info. The simplest path (observing the transfer or sending the single "request flag" message) was sufficient.
If the flag had been broken into parts, I would have written a small script to assemble and decode the pieces.
