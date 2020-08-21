# Metasploitable3 Vagrant


* Problems:
1. Warinng: Authentication failure. Retrying...
2. mount: unknown filesystem type 'vboxsf'

* Solutions:
1. add the following codes to Vagrantfile:

```ruby
config.ssh.username = "vagrant"
config.ssh.password = "vagrant"
```

and then run `vagrant reload`.

2. install vbox plugin to fix:

```sh
vagrant plugin install vagrant-vbguest
```


