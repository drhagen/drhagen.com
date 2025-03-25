---
icon: material/list-box
---

# CV

## Education

**Ph. D. in Biological Engineering**, 2014<br>
**Massachusetts Institute of Technology**, Cambridge MA

**B. S. in Biochemistry and B. A. in Economics**, 2008<br>
**Case Western Reserve University**, Cleveland OH<br>
Minor in Chemistry; graduated _summa cum laude_ with 4.0 GPA

**Ayersville High School**, Defiance OH<br>
Graduated valedictorian with 4.0 GPA

## Expertise

- Software Engineering: in particular, the data structures representing scientific workflows and the architecture to execute those workflows

- Statistics: in particular, information theory and Bayesian statistics with an appreciation for the importance of understanding the assumptions behind the techniques being applied

- Programming: Python, Matlab, and R; also Bash and C; past proficiency in Scala and C#

- Cloud computing: in particular, dockerized deployments and high performance computing in AWS

## Career History

### Software Engineer, *Applied BioMath / Certara*, 2017–present

**Director of Engineering**, Feb 2023–present

**Senior Principal Scientist**, Sep 2020–Feb 2023

**Associate Director of IT**, Apr 2020–Feb 2023

**Principal Software Engineer**, Jun 2017–Sep 2020

- Designed, architected, and managed the development of the Applied BioMath quantitative systems pharmacology (QSP) modeling platform, which was acquired by Certara in Dec 2023

- Oversaw the roll-out of the software platform to enthusiastic scientists as it replaced all internal usage of Matlab for systems pharmacology

- Replaced Django with FastAPI to provide a lower-maintenance backend for the platform

- Containerized the platform with Docker and deployed it to AWS using Terraform, eliminating reproducibility and scalability issues associated with running code on scientists' laptops

- Designed and implemented the parsing, validation, and compilation of four model text file formats to improve the productivity of modelers with progressively more expressive modeling domain-specific languages (DSLs)

- Directed the development of a core simulation engine in C, wrapping the SUNDIALS ODE solver library, achieving a 100x single-threaded performance improvement over the existing modeling tool, which enabled virtual patient population simulations on a large immuno-oncology consortium model that were previously intractable

- Designed and implemented a data grammar library in NumPy, exposing it via an API to replace specialized post-processing steps with customizable user-defined data pipelines

- Designed and implemented an expressions library to process models with arbitrary user-defined expressions, lower them to C code, compile them on-the-fly, and execute them as machine code for maximum performance

- Designed and implemented type checking, including units, across the API, ensuring consistency within models, across simulations, and through the data grammar pipelines, nearly eliminating unit errors from modeling reports

- Designed a custom job scheduler in asynchronous Python, reducing the latency of interactive jobs by 20x over Celery/Rabbit

### Data Scientist, *Merck*, 2014–2017

**Associate Principal Scientist**, May 2017–Jun 2017

**Senior Scientist**, Jul 2014–May 2017

- Improved the KroneckerBio toolbox to serve as platform for a QSP model of diabetes, which used literature, preclinical, and early data to predict outcomes of later trials and to reveal the desirable characteristics of backup compounds

- Used the KroneckerBio toolbox to develop a PBPK model of anacetrapib to investigate the source of the compound's long terminal half-life, supplying superior estimates of parameter uncertainty

- When the simulation platform for Keytruda crashed with a imminent filing, stepped in to reconstitute the workflow at the Linux command line over a couple of days, also bringing simulation time from 48 hours down to 20 minutes and analysis time from 40 hours to 4 minutes

- Served as technical lead on the development of a web-based workbench for NONMEM modeling and R analysis, ensuring traceability and reproducibility for modeling and simulation

- Designed and developed a preclinical compartmental fitting tool to automate the fitting and selection of a set of basic PK models to every batch of incoming study data with an appropriate design

- Designed and developed a Matlab tool for submitting arbitrary functions from a user's Matlab desktop to the Linux cluster, saving $17000 in license fees per user

- Designed and developed an internal R Shiny web application for converting SAS datasets to CSV to reduce the need for SAS installations and expertise

- Collaborated on the design and implementation of a R library for standardized plotting and reporting; designed and implemented a library to ease the checking of datasets for errors and for easily comparing two datasets

### Graduate Researcher, *Bruce Tidor lab, Massachusetts Institute of Technology*, 2009–2014

- Developed a novel method for computing the probability distribution on a set of discrete topologies according to a set of data using linearization as an approximation technique, showing that it gave a comparable answer to a gold-standard Monte Carlo method, while taking far less computational resources

- Demonstrated that linearization was an effective technique for approximating parameter uncertainty for the purpose of optimal experimental design, showing in a computational scenario, that the parameters of a biological network could be recovered from noisy data using a handful of optimally chosen experiments

- Developed and maintained KroneckerBio, the laboratory's systems biology toolbox written in Matlab, preforming simulation, parameter fitting, and statistical analysis of parameter and topology uncertainty.

- Managed a 2000-core high performance computing cluster

### Teaching Assistant, *Biological Engineering Department, Massachusetts Institute of Technology*, 2009

- In collaboration with co-TA Tim Curran, re-wrote the class tutorials, homework instructions, and implementation assignments of 20.420 Biomolecular Kinetics and Cellular Dynamics class, which by both the student and instructor accounts, greatly improved the learning to effort ratio of the out-of-class assignments, an effect that persists year after year as the material is reused by subsequent TAs

### Undergraduate Researcher, *Richard W Hanson lab, Case Western Reserve University*, 2005–2008

- Responsible for developing and testing hypotheses concerning the PEPCK-Cmus mice, a strain of super-active mice created in the lab
- Applied for and received funding for my own research project studying the effect of the transgene on metabolism and diabetes

## Publications

Diana H Marcantonio, Andrew Matteson, Marc Presler, John M Burke, David R Hagen, Fei Hua, Joshua F Apgar, "[Early Feasibility Assessment: A Method for Accurately Predicting Biotherapeutic Dosing to Inform Early Drug Discovery Decisions](https://www.frontiersin.org/articles/10.3389/fphar.2022.864768/full)", _Frontiers in Pharmacology_, 2022.

Bo Zheng, Lucia Wille, Karsten Peppel, David Hagen, Andrew Matteson, Jeffrey Ahlers, James Schaff, Fei Hua, Theresa Yuraszeck, Enoch Cobbina, Joshua F. Apgar, John M. Burke, John Roberts, Raibatak Das, "[A systems pharmacology model for gene therapy in sickle cell disease](https://ascpt.onlinelibrary.wiley.com/doi/10.1002/psp4.12638)", _CPT: Pharmacometrics & Systems Pharmacology_, 2021.

Rajesh Krishna, Ferdous Gheyas, Yang Liu, David R Hagen, Brittany Walker, Akshita Chawla, Josee Cote, Robert O Blaustein, David E Gutstein, "[Chronic Administration of Anacetrapib Is Associated With Accumulation in Adipose and Slow Elimination](https://onlinelibrary.wiley.com/doi/10.1002/cpt.700/abstract)", _Clinical Pharmacology and Therapeutics_, 2017.

David R Hagen and Bruce Tidor, "A Computational Experimental Design Method for Efficiently Reducing Topology Uncertainty", _unsubmitted_.

David R Hagen and Bruce Tidor, "[Efficient Bayesian Estimates for Discrimination among Topologically Different Systems Biology Models](https://pubs.rsc.org/en/Content/ArticleLanding/2014/MB/C4MB00276H)", _Molecular BioSystems_, 2015.

David R Hagen, "[Parameter and Topology Uncertainty for Optimal Experimental Design](./documents/DavidRHagenDoctoralThesis.pdf)", MIT Doctoral Thesis, 2014.

David R Hagen, Jacob K White, and Bruce Tidor, "[Convergence in parameters and predictions using computational experimental design](https://rsfs.royalsocietypublishing.org/content/3/4/20130008.full)", _Interface Focus_, 2013.

David R Hagen, Joshua F Apgar, David K Witmer, Forest M White, Bruce Tidor, "[Reply to Comment on 'Sloppy models, parameter uncertainty, and the role of experimental design'](https://pubs.rsc.org/en/content/articlelanding/2011/MB/c1mb05200d)", _Molecular BioSystems_, 2011.

Parvin Hakimi, Jianqi Yang, Gemma Casadesus, Duna Massillon, Fatima Tolentino-Silva, Colleen K Nye, Marco E Cabrera, David R Hagen, Christopher B Utter, Yacoub Baghdy, David H Johnson, David L Wilson, John P Kirwan, Satish C Kalhan, Richard W Hanson, "[Overexpression of the cytosolic form of phosphoenolpyruvate carboxykinase (GTP) in skeletal muscle repatterns energy metabolism in the mouse](https://www.jbc.org/content/282/45/32844.short)", _Journal of Biological Chemistry_, 2007.
