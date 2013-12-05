Prep Work
=========

Complete these items before the demo begins:



1.  Start with the PE 3.1.0 training vm, install a master and a node, sign the
    node certificate

2.  on the node, enable the public yum repos so we can find java

    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    puppet resource yumrepo base enabled=1 
    puppet resource yumrepo updates enabled=1
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

    //  Possible gotcha! - when we install and configure ntp, if the time is
    //  already wrong by a lot, it may mess up certificates when the time jumps.
    //  It’s probably a good idea to do a ntpdate before starting the demo, but
    //  be sure to remove ntpd again so we have more “wow” factor when
    //  installing it via a module.



Demo!
=====

Create a pug-environments git repo on github

1.  setup git/rename master to production to match puppet environments:

    1.  git clone git@github.com:adamcrews/pug-environments.git

    2.  Make at least 1 commit & push (the README.md is good)

    3.  git branch -m master production

    4.  git push origin production

    5.  On github change the default branch to production

    6.  git push origin :master

        1.  (https://github.com/githubtraining/feedback/issues/36)

2.  On the master - install r10k

    1. puppet module install zack-r10k

    2. classify master in the console with r10k
    
    3. Add https://github.com/adamcrews/pug-environments.git as the ‘remote’ class parameter for r10k

3.  On the master - /opt/bin/puppet/puppet agent -t

4.  In the checked out code

    1. mkdir {hieradata,manifests}

    2. Create Puppetfile

    3. Create hieradata/global.yaml

    4. Create manifests/site.pp that contains hiera_include(classes)

5.  On the master - /opt/puppet/bin/r10k deploy environment production -p --verbose

6.  On the master - Edit puppet.conf to insert:

    1.  /etc/puppetlabs/puppet/environments/$environment/modules  in the modulepath

    2.  manifest = /etc/puppetlabs/puppet/environments/$environment/manifests/site.pp 

    3.  On the master - Edit /etc/puppetlabs/puppet/hiera.yaml

    4.  set :datadir: to "/etc/puppetlabs/puppet/environments/%{::environment}/hieradata" 

    5.  add "fqdn/%{::fqdn}" to the search path

7.  On the master - restart pe-httpd

8.  On the master - puppet agent -t should work without errors

9.  In your source code create a ntp branch.

    1. Add ntp to the classes array in global.yaml

    2. add the key ntp::servers: with some servers to global.yaml

    3. add “mod ‘puppetlabs/ntp’, ‘3.0.0-rc1’” to Puppetfile

    4. git commit/push

10. On the master /opt/puppet/bin/r10k deploy environment ntp -p --verbose

11. On the master - puppet agent -t --environment=ntp

    1. This should setup ntp

12. Merge ntp branch to production

    1. git checkout production

    2. git merge ntp --no-ff

    3.  git push --all

13. Redeploy all r10k on the master - /opt/puppet/bin/r10k deploy environment production -p


Extra Credit - Minecraft server
===============================

1.  Create a minecraft branch and add minecraft classes

    1.  git branch -b minecraft

    2.  vi Puppetfile and add:

        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        mod 'branan/s3file', '1.0.1'
        mod 'puppetlabs/java', '1.0.1'

        mod 'minecraft',
          :git => 'https://github.com/adamcrews/puppet-module-minecraft.git',
          :ref => 'pug-demo'
        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

    3.  vi hieradata/fqdn/pug-node.puppetlabs.vm.yaml

        1.  Add minecraft to the classes array

    4.  git push origin minecraft

2.  On the master - /opt/puppet/bin/r10k deploy environment minecraft -p --verbose

3.  On the node - puppet agent -t --environment=minecraft

4.  Launch your minecraft client, and you should see the server

5.  Ooops… using an old mc server, let’s update it with puppet

    1.  add the key “minecraft::download_src: Minecraft.Download/versions/1.7.2/minecraft_server.1.7.2.jar” to the fqdn/$fqdn.yaml file.

    2.  commit, push, run r10k

    3.  puppet agent -t --environment=minecraft on the node

    4.  the mc server should restart automagically after 10 seconds, but this is buggy and you might need to run /sbin/service minecraft restart on the node.

    5.  Now you will see the updated server, and be able to connect with the mc client.

