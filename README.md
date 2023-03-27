# ROS-Neuro tutorial: How to create PPA repository on GitHub

This tutorial describes how to create a PPA repository on GitHub for ROS-Neuro. The following instructions have been used to create the [rosneuro/ppa](https://github.com/rosneuro/ppa) repository. The tutorial takes inspiration from [ppa-repo-hosted-on-github](https://assafmo.github.io/2019/05/02/ppa-repo-hosted-on-github.html).

## 0. PPA repository structure
We take as an example [rosneuro/ppa](https://github.com/rosneuro/ppa) repository with the following structure:
```
.
├── README.md
└── ubuntu
    ├── addkey_cmd
    ├── eegdev-plugins-free_0.3.0-1_amd64.deb
    ├── InRelease
    ├── libeegdev0_0.3.0-1_amd64.deb
    ├── libeegdev-dev_0.3.0-1_amd64.deb
    ├── Packages
    ├── Packages.gz
    ├── Release
    ├── Release.gpg
    ├── rosneuro-key.gpg
    ├── rosneuro.list
    ├── rosneuro-private-key.asc
    ├── ros-noetic-rosneuro-acquisition_1.0.1-0focal_amd64.deb
    ├── ros-noetic-rosneuro-acquisition-eegdev_2.0.0-0focal_amd64.deb
    ├── ros-noetic-rosneuro-acquisition-lsl_0.0.1-0focal_amd64.deb
    ├── ros-noetic-rosneuro-buffers_2.0.0-0focal_amd64.deb
    ├── ros-noetic-rosneuro-buffers-ringbuffer_1.0.0-0focal_amd64.deb
    ├── ros-noetic-rosneuro-data_1.0.0-0focal_amd64.deb
    ├── ros-noetic-rosneuro-filters_2.0.0-0focal_amd64.deb
    ├── ros-noetic-rosneuro-filters-blackman_1.0.0-0focal_amd64.deb
    ├── ros-noetic-rosneuro-filters-butterworth_1.0.0-0focal_amd64.deb
    ├── ros-noetic-rosneuro-filters-car_1.0.0-0focal_amd64.deb
    ├── ros-noetic-rosneuro-filters-dc_1.0.0-0focal_amd64.deb
    ├── ros-noetic-rosneuro-filters-flattop_1.0.0-0focal_amd64.deb
    ├── ros-noetic-rosneuro-filters-hamming_1.0.0-0focal_amd64.deb
    ├── ros-noetic-rosneuro-filters-hann_1.0.0-0focal_amd64.deb
    ├── ros-noetic-rosneuro-filters-laplacian_1.0.0-0focal_amd64.deb
    ├── ros-noetic-rosneuro-full_2.0.0-0focal_amd64.deb
    ├── ros-noetic-rosneuro-msgs_1.0.0-0focal_amd64.deb
    ├── ros-noetic-rosneuro-recorder_1.0.1-0focal_amd64.deb
    └── ros-noetic-rosneuro-visualizer_1.0.1-0focal_amd64.deb
 ```
 
## 1. Creating GitHub repository with rosneuro debian packages
Create a GitHub repo ```rosneuro/ppa```. Then go to https://github.com/rosneuro/ppa/settings, and under GitHub Pages select **Source** to be *Deploy from a branch* and in **Branch** select the branch *main*.
 
Now clone the repository and put all of your debian packages inside:
```
git clone "git@github.com:rosneuro/ppa.git"
cd ppa
cp rosneuro-package-*.deb .
 ```
 
## 2. Creating GPG Key
Install gpg and create a new key:
```
sudo apt install gnupg
gpg --full-gen-key
```
Use RSA:
```
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 1
```
RSA with 4096 bits:
```
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 4096
```
Key should be valid forever:
```
Please specify how long the key should be valid.
0 = key does not expire
<n> = key expires in n days
<n>w = key expires in n weeks
<n>m = key expires in n months
<n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) y
```
Enter your name and email:
```
Real name: My Name
Email address: ${EMAIL}
Comment:
You selected this USER-ID:
"My Name <my.name@email.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
```

At this point the gpg command will start to create your key and will ask for a passphrase for extra protection. 
```
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key **************** marked as ultimately trusted
gpg: directory '/home/assafmo/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/home/assafmo/.gnupg/openpgp-revocs.d/********************.rev'
public and secret key created and signed.

pub rsa4096 2023-03-27 [SC]
31EE74534094184D9964EF82B58FBB4C23247554
uid My Name <my.name@email.com>
sub rsa4096 2023-03-27 [E]
```
## 3. Creating the ```KEY.gpg``` file
Create the ASCII public key file KEY.gpg inside the git repository ppa:
```
gpg --armor --export "${EMAIL}" > /path/to/ppa/rosneuro-key.gpg
```
Note: The private key is referenced by the email address you entered in the previous step.

## 4. Creating the ```Packages``` and ```Packages.gz``` files
Inside the git repository ppa:
```
dpkg-scanpackages --multiversion . > Packages
gzip -k -f Packages
```

## 5. Creating the ```Release```, ```Release.gpg``` and ```InRelease``` files
Inside the git repository ppa:
```
apt-ftparchive release . > Release
gpg --default-key "${EMAIL}" -abs -o - Release > Release.gpg
gpg --default-key "${EMAIL}" --clearsign -o - Release > InRelease
```

## 6. Creating the ```rosneuro.list``` file
Inside the git repository ppa:
```
echo "deb [signed-by=/etc/apt/trusted.gpg.d/rosneuro-key.gpg] https://rosneuro.github.io/ppa ./" > rosneuro.list
```
This file will be installed later on in the user’s /etc/apt/sources.list.d/ directory. This tells apt to look for updates from your PPA in https://rosneuro.github.io/ppa.
