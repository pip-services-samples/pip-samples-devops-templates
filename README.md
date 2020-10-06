# Overview

This is a tutorial for Pip.Services Scripted Environment Templates. Environment can be divided to 3 parts:
- Management station
- Kubernetes cluster
- Database cluster
Same division approach used in Pip.Services Scripted Environment Templates. 

To get started you need to choose:
- Supported cloud (AWS, Azure, Google cloud) 
- Way of kubernetes deployment (single node by kubeadm or multi-node by cloud tools)
- Database (mongo, couchbase, etc.)

Full list of DevOps templates you can get on [Pip.Services templates](https://pipservices.webflow.io/extras/templates/overview) in section Scripted Environment Templates.

# Detailed example

Lets say we need 2 nodes kubernetes cluster hosted on AWS and a mongo database. In this case we need to download following templates:
- [Master template](https://github.com/pip-templates/pip-templates-env-master)
- [Management station](https://github.com/pip-templates/pip-templates-env-awsmgmt)
- [AWS kops cluster](https://github.com/pip-templates/pip-templates-env-awsk8s)
- [Mongo database](https://github.com/pip-templates/pip-templates-env-awsmongoatlas)

### Step 1. Download templates from github

Download master template
```sh
git clone git@github.com:pip-templates/pip-templates-env-master.git 
```

Download management station template
```sh
git clone git@github.com:pip-templates/pip-templates-env-awsmgmt.git
```

Download aws kubernetes template
```sh
git clone git@github.com:pip-templates/pip-templates-env-awsk8s.git
```

Download mongo template
```sh
git clone git@github.com:pip-templates/pip-templates-env-awsmongoatlas.git
```

Now you should have folder structure similar to:
```sh
$ ls -al
total 16
drwxr-xr-x 1 Dmitriy 197121 0 окт  5 11:41 ./
drwxr-xr-x 1 Dmitriy 197121 0 окт  5 11:13 ../
drwxr-xr-x 1 Dmitriy 197121 0 окт  5 11:41 pip-templates-env-awsk8s/
drwxr-xr-x 1 Dmitriy 197121 0 окт  5 11:40 pip-templates-env-awsmgmt/
drwxr-xr-x 1 Dmitriy 197121 0 окт  5 11:41 pip-templates-env-awsmongoatlas/
drwxr-xr-x 1 Dmitriy 197121 0 окт  5 11:13 pip-templates-env-master/

```

### Step 2. Merge templates

To merge template we need to have an empty folder which will be used as your environment management project directory. Lets create it

```sh
mkdir pip-templates-env-example
```

Now we need to copy master template to your environment management project directory:
```sh
cp -r pip-templates-env-master/* pip-templates-env-example/
```

Next step is copy management station template to your environment management project directory.
Copy root scripts:

```sh
cp pip-templates-env-awsmgmt/*.ps1 pip-templates-env-example/
```

Copy src and templates folders:

```sh
cp -r pip-templates-env-awsmgmt/src pip-templates-env-example/
cp -r pip-templates-env-awsmgmt/templates/ pip-templates-env-example/
```

Copy content of `pip-templates-env-awsmgmt/config/config.example.json` to `pip-templates-env-example/config/config.example.json`. Set aws related variables - *aws_access_id*, *aws_access_key*, *vpc*. Also you can change any variable in config depending on your requirements.

Now you need to merge aws kubernetes project, to do that please copy src and templates folders from aws kubernetes template to your environment management:

```sh
cp -r pip-templates-env-awsk8s/src pip-templates-env-example/
cp -r pip-templates-env-awsk8s/templates/ pip-templates-env-example/
```

Add content of all files with *.add* extenstion to similar files in your environment management project. For example in *pip-templates-env-awsk8s* you have *install_prereq_mac.sh.add*, you need to copy the content of this file to *install_prereq_mac.sh* in *pip-templates-env-example*. You need to do this step for following files in *pip-templates-env-awsk8s*: *create_env.ps1.add, destroy_env.ps1.add, install_prereq_mac.sh.add, install_prereq_ubuntu.sh.add, config.k8s.json.add*

Similar steps should be repeated for mongo template:

```sh
cp -r pip-templates-env-awsmongoatlas/src pip-templates-env-example/
cp -r pip-templates-env-awsmongoatlas/lib pip-templates-env-example/
```

Add content of all files with *.add* extenstion to similar files in your environment management project. You need to do it for following files in *pip-templates-env-awsk8s*: *create_env.ps1.add, destroy_env.ps1.add, install_prereq_mac.sh.add, install_prereq_ubuntu.sh.add, config.mongo.json.add*.

After all those step you have full environment management project.

### Step 3. Install management station

Environment management scripts implemented on powershell, management station should be created from local machine, so if you dont have powershell installed - use prerequisites scripts:

```sh
cd pip-templates-env-example
./install_prereq_ubuntu.sh
```

Do not forget to put id_rsa.pub key for management station in config folder, use same name which you set in config *mgmt_instance_keypair_name* variable.
To install management station you need to run *create_mgmt.ps1* script:

```sh
./create_mgmt.ps1 -c config/config.example.json
```

After management station successfully created you can get management station ip from resources file. Resources files stored in config folder and have similar name to used config, but it starts with resources. preffix. For example if you use *config/config.example.json* you will have *config/resources.example.json*. 

To connect to management station use *<mgmt_public_ip>* from *config/resources.example.json* file.

```sh
ssh -i config/<mgmt_instance_keypair_name>.pub ubuntu@<mgmt_public_ip>
```

### Step 4. Install environment

Environment should be created from management station, you need to connect to management station via ssh, example of ssh command you can view above.

On management station in home directory should be created folder *pip-templates-envmgmt* with your environment management.

```sh
cd ~/pip-templates-envmgmt
```

If this management station is newly created machine, which was never used by environment management scripts - it's probably required for installing prerequisits:

```sh
./install_prereq_ubuntu.sh
```

Create environment scripts includes creation of kubernetes cluster and mongo database.

```sh
./create_env.ps1 -c config/config.example.json
```

Created resources you can view of aws console or in *config/resources.example.json* file.
