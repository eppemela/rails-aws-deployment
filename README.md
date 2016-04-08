## Deploy a Rails App on Amazon AWS ElasticBeanStalk

First of all edit your ```config/database.yml``` with this settings for the production database:

```
production:
    <<: *default
    adapter: postgresql
    encoding: unicode
    database: <%= ENV['RDS_DB_NAME'] %>
    username: <%= ENV['RDS_USERNAME'] %>
    password: <%= ENV['RDS_PASSWORD'] %>
    host: <%= ENV['RDS_HOSTNAME'] %>
    port: <%= ENV['RDS_PORT'] %>
```

Then add puma to your Gemfile.

```
echo "gem 'puma' >> Gemfile "
bundle install
```

Initialize a new beanstalk app through the eb command.

In case you don't have the eb-cli install with:

```
brew install awsebcli
```
then


```
eb init
```
When prompted select 5 for Eu-central or 4 for eu-west **default region**.
Next, provide your access key and secret key so that the EB CLI can manage resources for you. Access keys are created in the AWS Identity and Access Management management console. If you don't have keys, see [How Do I Get Security Credentials?](http://docs.aws.amazon.com/general/latest/gr/getting-aws-sec-creds.html) in the Amazon Web Services General Reference.
Remember also to add *AWSElasticBeanstalkFullAccess* Policy to the IAM User you intend to use.

You'll then be prompted with some other questions:

```
EIt appears you are using Ruby. Is this correct?
(y/n): y

Select a platform version.
1) Ruby 2.3 (Puma)

Do you want to set up SSH for your instances?
(y/n): n
```

You should now see a **.elasticbeanstalk** directory in the root of your project with a **config.yml** file. Which was also automatically added into the **.gitignore** file for you.

```
branch-defaults:
  master:
    environment: null
    group_suffix: null
global:
  application_name: amazon_aws_deploy_test
  default_ec2_keyname: null
  default_platform: Ruby 2.3 (Puma)
  default_region: eu-central-1
  profile: eb-cli
  sc: git
```

Now let's setup our environments and run, in our example app_name is *amazon-aws-deploy-test*

```
eb create [app_name] --service-role aws-elasticbeanstalk-ec2-role
```

Check that everything went well with:
```
eb status
```
Now, lets add our **SECRET_KEY_BASE** environment variable by running **rake secret**

```
rake secret
```
In order to save this secret key in the elastic beanstalk env run **eb setenv [YOUR_SECRET_KEY_BASE]**
```
eb setenv SECRET_KEY_BASE=17ce7f0fa59ff47be20593462cd3dc0fb8f058d2f792d923b5f9d20959e018c87d5878dbeedd2b839571ab0aa79a0df65fae67c3750db3d322a30d6b67e538f1
```
Once the configuration is deployed successfull go to the [Elastic beanstalk console]( https://console.aws.amazon.com/elasticbeanstalk/home), click on the app name and then **configuration**, look for **data tier** and click on **create a new RDS database**, set the DB engine to **postgres** and create a **master username** and a **master password**, then click **Apply**.

Almost done. Create a folder named **.ebextensions** in the top level of our app with a **package.config** file in which we will add the **postgressql93-devel**
```
packages:
    yum:
      postgresql93-devel: []
```

And finally:

```
eb deploy
```
