# Plan your data management

Planning a research project can be at times overwelming especially if it is your first time approacing a similar task. 
While designing a project you need to consider several aspects about the data you would like to produce, such as: the most suitable kind of data for your research question, the amount of data you need and the best way to organize your data.
These are only a few essential aspects you might consider, there are of course many more, but one fundamental aspect that you should always keep in mind is **how to efficienty manage your data** in a way that the future you and other researchers can understand the development of your data, from conception to publication.

A good starting point would be to build a **Data Management Plan.**

## What is a Data Management Plan (DMP)?

A data management plan is a research document, taylored on your project, that, as the name suggests, helps you to organize your current and future data. It simply consists of a questionnaire that guides you through some of the aspects of the data life cycle.

Despite its name, a DMP should be considered as a living document that you can update throughout your project. Keeping your DMP up to date allows you to record essential information relative to your data, such as metadata and other sparse information that you might need in a later stage of the project (e.g. when writing a paper or a report for a funding agency). 

A DMP is increasingly required by funding agencies, since 2021 it is mandatory to provide a DMP for all the [European grants](https://ec.europa.eu/info/funding-tenders/opportunities/docs/2021-2027/common/agr-contr/general-mga_horizon-euratom_en.pdf), here you can find a DMP template by the [ERC](https://erc.europa.eu/sites/default/files/document/file/ERC_info_document-Open_Research_Data_and_Data_Management_Plans.pdf).

## How to create a DMP

There are many options to create a DMP, you could write your DMP by following templates from funding agencies, or you could use ad hoc online tools such as:

- [Argos](https://argos.openaire.eu/home)
- [dmponline](https://dmponline.dcc.ac.uk/)
- [Data stewardship wizard](https://ds-wizard.org/)
- [rdmo project](https://rdmo.aip.de/)
  
At Ipsy we are developing a simple, interactive prompt tool, called **Quest**, that will help you to create and customize your own DMP.

## Why should I use a DMP?

As previously mentioned, a DMP is incresingly becoming mandatory to acquire fundings from German and European agencies, but besides this incentive it provides some other real advantages:

- It helps you to critically assess different aspects of your data that you could easily overlook.
- It helps you to think of your data organically.
- It forces you, at an early stage of your project, to create documentation and proper metadata.
- If properly updated during the project life, you can publish the DMP together with your data.

!!! Danger "Start writing documentation"
    Documenting your data only at the end of your project is not advisable. It forces you to rely on your fallible memory, on sparse information and it is extremely error prone and time consuming


## Planning your project at Ipsy

At Ipsy, we strive for open and reproducible science, to achieve this goal we have implemented a basic **research project workflow** that can help you manage your data with very little effort. 

In order to follow the project workflow we suggest you to use the **cluster** for data analysis and data storage. The project workflow consists of three simple steps:

1. Fill in a brief **questionnaire** that help us to create some intial metadata for your project. Further information about the questionnaire can be found [here] 
[here]: ../../../cluster/cecile/data/#how-to-create-a-project-on-cecile

2. Once the questionnaire is properly filled in, we create a dedicated **project** which comes with a predefined folder structure (to have detailed information about the structure, please refer to the specific [project structure section].
[project structure section]: ../../../cluster/cecile/data/#project-structure

1. Once your raw data are moved into the predefined structure they need to be converted into the **BIDS** format (see [BIDS]) by using the tools available in the cluster.
[BIDS]: ../research-data-management/bids.md

