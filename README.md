# lbg-guide
Basic usage tips for the LBG cluster at UNC-CH

## What is LBG?
Scientific computing at UNC-CH happens mostly on the [Longleaf Cluster](https://its.unc.edu/research-computing/longleaf-cluster/). Additionally, the [Lineberger Cancer Center](https://unclineberger.org/) also has its own dedicated cluster called LBG, which is administered by the [Lineberger Bioinformatics Core](https://lbc.unc.edu/). This unofficial repository will hopefully serve as a basic guide for LCCC researchers to get started using LBG. 

## Access

To connect to any LBG login node: 

```sh
ssh <your_onyen>@login.bioinf.unc.edu
```

If you want to maintain a tmux session and reconnect to it later, it is better to instead connect to a particular login node:

```sh
ssh <your_onyen>@vm-login-2-0.bioinf.unc.edu
```

