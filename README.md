# Motivation

* I used to store my credentials in a service which is used to take notes.
* One day, I opened another service from the same service provider and my credentials started appearing as suggestions.
* That day, I realized that password managment is not something to be taken lightly especially when you start to take responsibility and there are people who depend on you.

# Mental Model

* Instead of using a "third-party" service to store your credentials, keep them locally.
* It does not matter how "big" a service provider is, you are never 100% sure about safety online.
* With that said, people have multiple devices on which they need their credentials. Hence the need to keep a centralized system which stores the password locally and a process through which you can get those passwords to your other machines.
* As someone great once said, _If you have a problem, the terminal is the solution_ (this NEEDS to be said if someone hasn't said this already)
* Since I already have a home server setup running on my late 2011 MacBook Pro, I can use that machine as my central node to store the credentials.
* Store the credentials in a file.
> Note: I will not share the format I store my credentials in due to safety reasons.
* Encrypt the file using `gpg` and suitable encryption algorithm
* Delete the original text file using `shred` in order to securely delete and override the physical memory page(s).

## Using Credentials

* Now comes the problem of sending the credentials to other machines
> Note that I am running tailscale on all my machines due to the CG-NAT problem with Airtel ISP and in order to access my machines from anywhere securely.
* The gist of the solution is to run a command on the central node remotely using `ssh`
* Now what that command is, is the main point

### Solution 1

```
ssh [user]@[addr] "gpg -d [file] | grep -i [keyword] | awk '{print $x}'" | [copy]
```

* We decrypt the file on the central node
* Find the specific service using the keyword
* Select the column/part where the password is stored
* And then copy that word into the local system's clipboard using `xclip` in case of linux or `pbcopy` in case of MacOS

#### Main Problems

* The problem with this solution is that the decrypted password is sent over the network from the central node to the local machine
* Although ssh is very secure, we would like to minimize the risk as much as possible

### Solution 2

```
ssh [user]@[addr] "cat [file]" | gpg -d | grep -i [keyword] | awk '{print $x}' | [copy]
```

* We only pass the encyrpted file over the network which no one can decrypt with the current hardware and in logical time without the key
* Then we decrypt it locally and then follow the same steps as earlier to retrieve and copy the password to clipboard

#### Main Problems

* When the decryption happens, it exposes all of the passwords to the pipeline
* Furthermore the decryption time also increases

#### Solution 3

* We divide each password to a specific .gpg file
* This results in the decryption only hapenning for the service whose password we want
* This leads to more security and faster access time
* So, we ask for the list of files from the central node
* Pick one using a program like fzf
* Then get the encrypted file's content over ssh
* Decrypt the file locally and extract the password

```
files=$(ssh [user]@[addr] "ls folder/")
file=$(echo "$files" | fzf)
ssh [user]@[addr] "cat $file" | gpg -d | [extract password]
```
