# About this lab

This lab contains a series of exercises illustrating topics of automation and IaC deployment in AWS.

## Prerequisited
In order to use this lab you will need the following:
* git (https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* aws cli - command line interface (https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

Once you haev installed the **aws cli** you need to configure your credentials as per this ducument (https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)

You can verify that your installation and credentials have been setup corrrectly by executing the following command:
```
aws sts get-caller-identity
```

This should produce outpur similar to the one below:

```
{
    "UserId": "AROAY3SZJCHZXVWXBFX2Q:test",
    "Account": "123454645667",
    "Arn": "arn:aws:sts::123454645667:assumed-role/Administrator/test"
}
```

## Using the exercises

In order to use the exercises you will need to clone this repository to your local machine using following command:

```
git clone https://github.com/marcinkaluza/sil-automation.git
```

All exercises contain their detailed instructions:

* [Exercise 1](/Exercise1/README.md)
* [Exercise 2](/Exercise2/README.md)
* [Exercise 3](/Exercise3/README.md)
* [Exercise 4](/Exercise4/README.md)
* [Exercise 5](/Exercise5/README.md)
