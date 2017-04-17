# Test a local version of Juju

These are the steps that I used to test my own compiled Juju in my environment.

## Creating local streams

Juju tools are named using the version, series and architecture. In my case, I was using version 2.2-beta1 and 
I needed xenial series for the controller and opensuseleap (the new series I introduced).
Tools should be stored in a path related to the strem (`devel` in my case, even the agent-stream was configured to `released`)

```bash
mkdir $HOME/simplestreams/tools/devel
cd simplestreams/tools/devel
cp $GOPATH/bin/jujud .
tar zcvf juju-2.2-beta1-opensuseleap-amd64.tgz jujud
tar zcvf juju-2.2-beta1-xenial-amd64.tgz jujud
rm jujud
```

Using `juju-metadata` we can generate the metadata for this stream:

```bash
cd $HOME
juju-metadata generate-tools -d simplestreams/ --show-log --clean --stream devel
```

We need a web server for publishing the tools. I used `nginx`:
```bash
sudo rm -rf /var/www/html/tools
sudo cp -r $HOME/simplestreams/tools /var/www/html
sudo chmod -R 755 /var/www/html/tools
```

And finally, we can bootstrap juju and we need to indicate the URL of our tools.

```bash
juju bootstrap --config agent-metadata-url="http://10.190.55.1/tools/" localhost lxd-opensuse
```

note that `10.190.55.1` is the IP address of my compute host in the `LXD` subnet.
