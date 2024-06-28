# Getting started

## What is a computational cluster?

A computational cluster is a collection of interconnected computers (see image below), called **compute nodes**, accessable and operable via a single computer called the **head node**. The strength of a cluster comes from the collective interplay among the nodes which allows them to work as a unique entity able to perform computationally expensive parallel tasks (e.g. analyses, modelling etc.) otherwise difficult and in some case impossible to achieve by a single computer. A cluster can be accessed remotely from your computer (a client in the image) through the head node. 

<figure markdown="span">
  ![Cluster](images/cluster.png){ width="500" }
  <figcaption></figcaption>
</figure>

## Why should I use a cluster instead of my own computer?

- **Time**: A cluster allows you to perform computationally expensive tasks in a reduced amount of time with respect to your laptop.
  
- **Parallelization**: A typical project for cognitive neuroscience requires the analysis of group of subjects or animals, which can often take long time, by leveraging on the multiple nodes, you can perform virtually at the same time the analsys of different subjects.
   
- **Security and backups**: The cluster ensures that your data are constantly backed up and protected, thus allowing also to store and analyze sensitive data (e.g. patient data).
  
- **Reproducible environment**: The cluster software set-up guarantees a stable environment which contains all the tools you need for your analyses that can be easily reproduced outside the cluster itself.

## What do you need to do to use the cluster?

- **An account:** You simply need an account, please contact manuela.kuhn at ovgu.de to obtain one.

- **Shell scripting:** If you prefer to work on the cluster via shell (this is canonical way), but you have no experience with shell scripting, we recommend this straighforward [shell tutorial](https://swcarpentry.github.io/shell-novice/). You do not need to master it to use the cluster, but you can use the following list to learn the basic concepts:
  
  -  where your are and how to navigate through folders --> `pwd`, `cd`
  -  how to list content (folders/files) --> `ls` 
  -  how to create and remove folders/files --> `mkdir`, `touch`, `rm`
  -  how to copy folders/files --> `rsync`, `cp`
  -  how to move and rename folders/files --> `mv`
  -  how simple permissions work and how to modify them --> `chmod`
  -  how to write a simple shell script and execute it</b>  


- **An editor:** Choose an editor among the available ones: **Nano** (the most user friendly), **Vim**, **Emacs**.

## Clusters at Ipsy:

At ipsy we currently have two clusters: the new one, [Cecile], named after the French neurologist Cécile Vogt, and the old one, [Medusa], named after the Greek mithological figure.

[Cecile]: cecile/access
[Medusa]: medusa/access