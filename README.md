# Chat-PGP-P2P
The goal of this project is to create a P2P chat application secure.
It will use the PGP concept to secure its communications.

## 1. Local architecture
The application can be defined on two processes :
  - a core process that manage all interactions with other contacts;
  - an GUI process that display and receive order from the user.
  
![alt tag](https://docs.google.com/drawings/d/1QxrPv2GBXfWVg0UZ9v2EOpYFAVSnTN4NwQdItwjrMog/pub?w=1151&h=592)

### 1.A. The core process
The core process is a background process working like a server.

When this process is launch, it open a UDP listener to add new contacts and a TCP listener to allow discussion requests. 
It will also open other TCP listeners for each new discussion created.

It will listen each listener to get new contacts and new messages.
It will also check the GUI process for new message to send and for new discussion to create.
Only non-block sockets is used to allow the core process to listen two or more listeners. 

It memorize each opened TCP connections, each generated/shared key and each messages send and received from/to other contacts.
That make a good separation between the GUI interface and the working process.

### 1.B. The GUI process
The GUI process has an interface to communicate with the core process.
A web interface, a command application, or a GUI application could be used independantly to the core process.

## 2. Protocol
Each application have several couples of public/private keys :
 - one in general to begin a discussion and to authentificate every messages from this contact;
 - one for each discussion to separe each of them.

The client and server have many roles.

### 2.A. Who's there (UDP mode)
First of them is to find neighbours arround.
For that, there are two ways:
 - the core process send an UDP broadcast request to be seen by other contacts and wait for responses;
 - when the core process receive an UDP request from other contacts.
 
![alt tag](https://docs.google.com/drawings/d/1Ztd4E9MDaGBN20A3rO09nqmKqa9q4oCFvaX7XVc6CbI/pub?w=2020&h=597)
 
For this recognition task, the general couple of keys will be used to authentificate each contact when discussion request is made.
The UDP broadcast message contains the owner public key and the UDP response message contains the distant contact public key.
Neither of theses messages are crypted.

This step only help the application to be seen by others and is done regulary.

### 2.B. Let's talk together (TCP mode)
After some UDP messages exchanged, a list of contact should be available and the user can send a request to open a discussion.

![alt tag](https://docs.google.com/drawings/d/18N_1828-j1j96WrsqvAtdIF_4W71BCFZOvxV8uAnnlI/pub?w=1014&h=959)

As long as new couples of keys aren't been created in both sides and publics keys aren't been communicated, all messages will be crypted by using the general couple of keys of each application.

When an user wants to open a conversation to another, it create an TCP tunnel to communicate. 
But before that, a secret password have to be transmitted by other secure transmission way to recognize the distant user behind the application.

Then a couple of public/private key specific to this discussion is created and the user application send an request to the distant application including this public key and the prompted password.

Depending to the secret password received, the distant user has the choice to keep the connection or to refuse it.
If it is the right password, it send its public key created for this discussion. In the other case, it will close the connection.

At the end of this step a discussion between the two users is created. They can now 

### 2.C. What's up (TCP mode)
The discussion created, both users can send signed and crypted messages by using its private key and the distant user's public key.

![alt tag](https://docs.google.com/drawings/d/13NqTWwHt7ozKSw3iij0Wb2Qfr89rIuoA4ILcz4T9pOY/pub?w=1009&h=486)

## 3. Peer to peer on internet
The main problem of this project is probably to find contacts arround us.
When the application is only used into an internal network, that isn't problematic : we can send a broadcast request like explain in the part 2.A.
But when we want to use it outside, a broadcast request cannot be done like it. Firewall usually bloc this kind of requests.

Even if broadcast requests is impossible to find some neightbors, a UDP connection with them still be possible.
By using a specific IP address, two neighbours can be join.
Then a discussion could be created between the two users like descript at part 2.B.

That why, we need to find other way to find some neightbors by getting some IP address.

### 3.A. Using a file
One solution could be a list of contacts which contains IP address of potential neighbours.
This kind of list is already used by bitorrent applications which use a torrent file to connect to others.

In our case, instead of send one UDP brocast request, the application send the same UDP request on each contact from the list.
This list can also be completed with the neighbours' list content to get more contacts. For that, when the client send a request to another user, the distant server send a list of its contacts in addition to its public key.

### 3.B. Using a third-party server
Another solution could be a third-party server which contains a regulary updated list of contacts.
This kind of system is already used by applications such as Facebook or Twitter for some functionalities.

In our case, the application can connect to this third-party server to get a list of contacts.
Then it send the UDP request on each contact.
Of course, by receiving the request of contacts, this third-party server should add the requested user to its list of contacts for few times.

The application should send this kind of request regulary to get more contacts. By this way, it also declare to the server that it still available to discuss with others.
