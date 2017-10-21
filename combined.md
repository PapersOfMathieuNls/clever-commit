---
title: "CLEVER: Combining Code Metrics with Clone Detection for Just-In-Time Fault Prevention and 
Resolution in Large Industrial Projects"
bibliography: config/library.bib
abstract:  "Abstract goes here"
author: 
- name: Mathieu Nayrolles
  affiliation: La Forge Research Lab, Ubisoft
  location: Montréal, QC, Canada
  email: mathieu.nayrolles@ubisoft.com
- name: Abdelwahab Hamou-Lhadj
  affiliation: ECE Department, Concordia University
  location: Montréal, QC, Canada
  email: wahab.hamou-lhadj@concordia.ca
csl: config/ieee.csl
classoption: 
- 10pt
- conference
keyword: 
- Bug Prediction
- Risky Software Commits
- Clone Detection
- Software Maintenance
---

\section{Introduction}\label{sec:introduction}

\IEEEPARstart{A}{utomatic} prevention and resolution of faults is an important research topic in the field of software maintenance and evolution. A particular line of research focuses on the problem of preventing the introduction of faults by detecting risky commits (commits that may potentially introduce a fault in the system) before reaching the central code repository. We refer to this as just-in-time fault detection/prevention.  Recent techniques such as the work of Rosen et al. [@Rosen2015] rely on building  models from historical commits using code and process metrics (e.g., code complexity, the experience of the developers, etc.) as the main features. These models are later used to classify new commits as risky or not. The problem with these techniques is that they generate high false positive rates. In addition, tools that implement these techniques such as Commit-guru [@Rosen2015] do not provide sufficient insights to developers on how to fix the risky commits. They simply return measurements that are often difficult to interpret by developers. These have also been applied to single projects only, despite the fact that  many projects share dependencies due to the reuse of libraries, which make them vulnerable to similar faults. In addition, existing techniques are mainly validated using open source systems. Their effectiveness when applied to industrial systems has yet to be shown.  

In this paper, we propose an approach, called CLEVER (Combining Levels of Bug Prevention and Resolution techniques), that relies on a two-phase process for intercepting risky commits before they reach the central repository. The first phase consists of building a metric-based model to assess the likelihood that an incoming commit is risky or not. This is similar to existing approaches, except that our model is built using commits from multiple inter-related projects. In the next phase, CLEVER uses clone detect to compare code blocks extracted from risky commits (detected in the first phase) with those of known fault-introducing commits. CLEVER  does not detect risky commits  by only comparing them to commits of a single project but also to those belonging to other projects that share common dependencies.  As we will show in the case study, this second phase reduces the number of false positives. Another advantage of  CLEVER is that it uses commits that are used to fix previous fault-introducing commits to guide the developers on how to improve the risky commits. This way, CLEVER goes one step further than Commit-guru (and similar techniques) by providing developers with a potential fix for their risky commits.
 
CLEVER was developed in collaboration with software developers from Ubisoft La Forge. Ubisoft is one of the world's largest video game development companies specializing  in the design and implementation of high-budget video games. Ubisoft software systems are highly coupled with  millions of files and commits,  developed and maintained by  more than 8,000 developers scattered across 29 locations in six continents.

When applied to 12 Ubisoft systems, CLEVER  detects risky commits with 79% precision and 65% recall, which outperforms the performances of Commit-guru, which  is 66% precision and 63% recall when applied to the same dataset. In addition, more than XX% of the proposed fixes were judged highly relevant by Ubisoft software developers, making CLEVER an effective and practical approach for the detection and resolution of risky commits.

The remaining parts of this paper are organised as follows. In Section \ref{sec:relwork}, we present related work. Sections \ref{sec:CLEVERT},  \ref{sec:exp} and \ref{sec:result} are dedicated to the CLEVER approach, the case study setup, and the case study results.
Then, Sections \ref{sec:threats} and \ref{sec:conclusion} present the threats to validity and a conclusion accompanied with future work.

# Related Work {#sec:relwork}

The work most related to ours come from two main areas, work that aims to predict future defects in files, modules and changes and work that aims to propose or generate patches for buggy software.

## File, Module and Risky Change Prediction

The majority of previous file/module-level prediction work used code or process metrics.
Approaches using code metrics only use information from the code itself and do not use any historical data. Chidamber and Kemerer published the well-known CK metrics suite [@Chidamber1994] for object oriented designs and inspired Moha *et al.* to publish similar metrics for service-oriented programs [@Moha]. Another famous metric suite for assessing the quality of a given software design is Briand's coupling metrics [@Briand1999a].

The CK and Briand’s metrics suites have been used, for example, by Basili *et al.* [@Basili1996], El Emam *et al.* [@ElEmam2001], Subramanyam *et al.* [@Subramanyam2003] and Gyimothy *et al.* [@Gyimothy2005] for object-oriented designs. 
Service oriented designs have been far less studied than object oriented design as they are relatively new, but, Nayrolles *et al.* [@Nayrolles; @Nayrolles2013a], Demange *et al.* [@demange2013] and Palma *et al.* [@Palma2013] used Moha *et al.* metric suites to detect software defects.
All these approaches, proved software metrics to be useful at detecting software fault for object oriented and service oriented designs, respectively. More recently, Nagappan *et al.* [@Nagappan2005; @Nagappan2006] and Zimmerman *et al.* [@Zimmermann2007; @Zimmermann2008] further refined metrics-based detection by using statical analysis and call-graph analysis.

Other approaches use historical development data, often referred to as process metrics. Naggapan and Ball [@Nagappan] studied the feasibility of using relative churn metrics to prediction buggy modules in the Windows Server 2003. Other work by Hassan *et al* and Ostrand *et al* used past changes and defects to predict buggy locations (e.g., [@Hassan2005], [@Ostrand2005]). Hassan and Holt proposed an approach that highlights the top ten most susceptible locations to have a bug using heuristics based on file-level metrics [@Hassan2005]. They find that locations that have been recently modified and fixed locations are the most defect-prone. Similarly, Ostrand *et al.* [@Ostrand2005] predict future crash location by combining the data from changed and past defect locations. They validate their approach on industrial systems at AT&T. They showed that data from prior changes and defects can effectively defect-prone locations for open-source and industrial systems. Kim *et al.* [@Kim2007a] proposed the bug cache approach, which is an improved technique over Hassan and Holt's approach [@Hassan2005]. Rahman and Devanbu found that, in general, process-based metrics perform as good as or better than code-based metrics [@rahman2013].

Other work focused on the prediction of risky changes. Kim et al. proposed the change classification problem, which predicts whether a change is buggy or clean [@SunghunKim2008]. Hassan [@Hassan2009] used the entropy of changes to predict risky changes. They find that the more complex a change is, the more likely it is to introduce a defect. Kamei *et al.* performed a large-scale empirical study on change classification [@Kamei2013]. They aforementioned studies find that size of a change and the history of the files being changed (i.e., how buggy they were in the past) are the best indicators of risky changes.

Our work shares a similar goal to works on the prediction of risky changes. However, CLEVERT takes a different approach in that it leverages dependencies of a project to determine risky changes.

## Automatic Patch Generation

Since CLEVER not only flags risky changes but also provides developers with fixes that have been applied in the past, automatic patch generation work is also related. Pan *et al.* [@Pan2008] identified 27 bug fixing patterns that can be applied to fix software bugs in Java programs. They showed that between 45.7 - 63.6\% of the bugs can be fixed with their patterns. Later, Kim *et al.* [@Kim2013] generated patches from human-written patches and showed that their tool, PAR, successfully generated patches for 27 of 119 bugs. Tao *et al.* [@tao2014automatically] also showed that automatically generated patches can assist developers in debugging tasks. Other work also focused on determining how to best generate acceptable and high quality patches, e.g. [@Dallmeier; @le2012systematic], and determine what bugs are best fit for automatic patch generation [@le2015should].

Our work differs from the work on automated patch generation in that we do not generate patches, rather we use clone detection to determine the similarity of a change to a previous risky change and suggest to the developer the fixes of the prior risky changes.

# The CLEVER Approach {#sec:CLEVERT}

Figures \ref{fig:CLEVERT1}, \ref{fig:CLEVERT3} and \ref{fig:CLEVERT2} show an overview of the CLEVER approach, which consists of two parallel processes. 

In the first process (Figures \ref{fig:CLEVERT1} and \ref{fig:CLEVERT3}), CLEVER manages events happening on project tracking systems to extract defect-introducing commits and commits used to  provide the corresponding fixes. For simplicity reasons, in the rest of this paper, we refer to commits that are used to fix defects as _fix-commits_. 
We use the term _defect-commit_ to mean a commit that introduces a fault. 

The project tracking component of CLEVER listens to bug (or
issue) closing events of Ubisoft projects. 
Currently,CLEVER is tested with 12 large projects within Ubisoft. 
These projects share many dependencies. 
We clustered the projects based on their dependencies with the aim to improve the accuracy of CLEVER.
This clustering step is important in order to identify faults that may exist due to dependencies, while enhancing the quality of the proposed fixes. 
Applying CLEVER to projects that are not related to each other may turn to be ineffective. 

\input{tex/approach}

In the second process (Figure \ref{fig:CLEVERT2}), CLEVER intercepts incoming commits before leaving the developers' workstation using the concept of pre-commit hooks. A pre-commit hook is a script that is executed at commit-time and it is supported by most major code versioning systems such as Github. There are two types of hooks: client-side and server-side. Client-side hooks are triggered by operations such as committing and merging, whereas server-side hooks run on network operations such as receiving pushed commits. These hooks can be used for different purposes such as checking compliance with coding rules, or the automatic execution of unit tests. A pre-commit hook runs before a developer specifies a commit message.

Ubisoft's developers use pre-commit hooks for all sorts of reasons such as identifying the tasks that are addressed by the commit at hand, specifying the reviewers who review the commit, and so on. Implementing this part of CLEVER as a pre-commit hook is an important step towards the integration of CLEVER with the workflow of developers at Ubisoft. The developers  do not have to download, install, and understand additional tools in order to use CLEVER. 

Once the commit is intercepted, we compute code and process metrics associated with this commit. The selected metrics are discussed further in Section \ref{sec:offline}. The result is a feature vector (Step 4) that is used for classifying the commit as _risky_ or _non-risky_. 

If  the commit is classified as _non-risky_, then the process stops, and the commit can be transferred from the developer's workstation to the central repository. _Risky_ commits, on the other hand, are further analysed in order to reduce the number of false positives (healthy commits that are detected as risky). We achieve this by first extracting the code block that is modified by the developer in order to apply the commit and then comparing it to code blocks of known fault-introducing commits.


## Clustering Project Repositories {#sec:clustering}

We cluster projects according to their dependencies. The rationale is that projects that share dependencies are most likely to contain defects caused by misuse of these dependencies. In this step, the project dependencies are analysed and saved into a single NoSQL graph database as shown in Figure \ref{fig:CLEVERT3}. A node corresponds to a project that is connected to other projects on which it depends. Dependencies can be _external_ or _internal_ depending on whether the products are created in-house or supplied by a third-party. For confidentiality reasons, we cannot reveal the name of the projects involved in the project dependency graph. We show the 12 projects in yellow color with their dependencies in blue color \ref{fig:dep-graph}. The resulting partitioning is shown in \ref{fig:network-sample}. 

\input{tex/dependencies}

Internal dependencies are managed within the framework of a single repository, which makes their automatic extraction possible. The dependencies could also be automatically retrieved if the projects use a dependency manager such as Maven. 

\input{tex/network-sample}

Once the project dependency graph is extracted, we use a clustering algorithm to partition the graph. To this end, we choose the Girvan–Newman algorithm [@Girvan2002; @Newman2004], used to detect communities by progressively removing edges from the original network. Instead of trying to construct a measure that identifies the edges that are the most central to communities, the Girvan–Newman algorithm focuses on edges that are most likely "between" communities. This algorithm is very effective at discovering community structure in both computer-generated and real-world network data [@Newman2004]. Other clustering algorithms can also be used. 


## Building a Database of Code Blocks of Defect-Commits and Fix-Commits {#sec:offline}

To build our database of code blocks that are related to defect-commits and fix-commits, we first need to identify the respective commits. Then, we extract the relevant blocks of code from the commits.

**Extracting Commits:** CLEVER listens to issue closing events happening on the project tracking system used at Ubisoft. Every time an issue is closed, CLEVER retrieves the commit that was used to fix the issue (the fix-commit) as well as the one that introduced the defect (the defect-commit). To link fix-commits and their related issues we implemented the SZZ algorithm presented by Kim et al. [@Kim2006c]. 


**Extracting Code Blocks:**  Algorithm \ref{alg:extract} presents an overview of how extract blocks. This algorithm receives as arguments, the changesets and the blocks that have been previously extracted. 
Then, Lines 1 to 5 show the $for$ loop that iterates over the changesets. 
For each changeset (Line 2), we extract the blocks by calling the $~extract\_blocks(Changeset~cs)$ function. 
In this function, we expand our changeset to the left and to the right in order to have a complete block.

\input{tex/extract}

As depicted by the diff below (not from Ubisoft), changesets contain only the modified chunk of code and not necessarily complete blocks. 

```diff
@@ -315,36 +315,6 @@
int initprocesstree_sysdep
(ProcessTree_T **reference) {
	mach_port_deallocate(mytask,
	  task);
}
}
- if (task_for_pid(mytask, pt[i].pid,
-  &task) == KERN_SUCCESS) {
-   mach_msg_type_number_t   count;
-   task_basic_info_data_t   taskinfo;
```

Therefore, we need to expand the changeset to the left (or right) to have syntactically correct blocks. 
We do so by checking the block's beginning and ending with parentheses algorithms [@bultena1998eades]. 

## Building a Metric-Based Model

The metric-based model is the first filter of classification of CLEVER.
To build our metric based model, we inspired ourselves from Commit-guru [@Rosen2015].
Commit-guru classifies commits using a list of keywords proposed by Hindle *et al.* [@Hindle2008] to classify commit in terms of _maintenance_, _feature_ or _fix_.
Then it uses the SZZ algorithm find the bug introducing commit linked to the fix-commit.
For each bug introducing commit, the following code metric are computed: _la_ (lines added), _ld_ (lines deleted), _nf_ (Number of modified files), _ns_ (Number of modified subsystems), _nd_ (number of modified directories), _en_ (distriubtion of modified code across each file), _lt_ (lines of code in each file (sum) before the commit), _ndev_ (the number of developers that modifed the files in a commit), _age_ (the average time interval between the last and current change), _exp_ (number of changes previously made by the author ), _rexp_ (experience weighted by age of files (1 / (n + 1))), _sexp_ (previous  changes made by the author in the same subsystem), _loc_ (Total modified LOC across all files), _nuc_ (number of unique changes to the files).
Then, a statistical model is built using the metric values of the bug-introducing commits. 
Using a linear regression, Commit-guru is able to predict if new commits are _risky_ by computing their code metrics.

CLEVER on the other hand, uses internal project tracking system. 
In other words, it only classifies commits as introducing a defect if they are the root cause of a fix linked to a crash in the internal project tracking system (JIRA). Project tracking system allows one to report unexpected system behavior and managers can assign them to developers.
Using the internal pre-commit hook, developers must link every commit to a give task #ID. If the task #ID entered by the developer matches a bug or crash report within the project tracking system, then we perform the SCM blame/annotate function on all the modified lines of code for their corresponding files on the fix-commit's parents. This returns the commits that previously modified these lines of code, and are flagged as defect-commits.
We had to rewrite Commit-guru in GoLang for performance and internal reason and adapted it to classify commit purpose (_maintenance_, _feature_ or _fix_) using the project tracking system). 
For the actual classification, we do not use a linear regression but the random forest algorithm. The random forest algorithm proved to be more efficient as described in section \ref{sec:result}.

## Comparing Code Blocks  {#sec:online}

Each time a developer makes a commit, CLEVER intercepts it using a pre-commit hook and classifies it as _risky_ or _sane_.
If the commit is classified as _risky_ by the metrics-based classifier, then, we extracts the corresponding code block (in a similar way as in the previous phase), and compares it to the code blocks of historical defect-commits. 
If there is a match, then the new commit is deemed to be risky. A threshold $S$ is used to assess the extent beyond which two commits are considered similar. 

To compare the extracted blocks to the ones in the database, we resort to clone detection techniques, more specifically, text-based clone detection techniques. This is because lexical and syntactic analysis approaches (alternatives to text-based comparisons) would require a complete program to work, i.e., a program that compiles.  
In the relatively wide-range of tools and techniques that exist to detect clones by considering code as text [@Johnson1993;  @Johnson1994; @Marcus; @Manber1994; @StephaneDucasse; @Wettel2005] we picked to use the NICAD clone detector because it is freely available and is a text-based clone detector [@Cordy2011].
We improve on NICAD to give it the ability to precess blocks that comes from commit-diffs.
Indeed, NICAD, in its vanilla version is can only process syntactically correct code and commit-diffs are, by definition, snippet that represents modified regions of a given set of files.

NICAD can detect Types 1, 2 and 3 software clones [@CoryKapser]. Type 1 clones are copy-pasted blocks of code that only differ from each other in terms of non-code artefacts such as indentation, whitespaces, comments and so on.  Type 2 clones are blocks of code that are syntactically identical except literals, identifiers, and types that can be modified. Also, Type 2 clones share the particularities of Type 1 about indentation, whitespaces, and comments. Type 3 clones are similar to Type 2 clones in terms of modification of literals, identifiers, types, indentation, whitespaces, and comments but also contain added or deleted code statements. CLEVER detects Type 3 clones since they can contain added or deleted code statements, which make them suitable for comparing commit code blocks.

NICAD uses a pretty-printing strategy from where statements are broken down into several lines [@Iss2009].

The pretty-printing allows us to detect Segments 1 and 2 as a clone pair, as shown by Table \ref{tab:pretty-printing}, because only the initialization of $i$ changed. This specific example would not have been marked as a clone by other tools we tested such as Duploc [@StephaneDucasse]. 

\input{tex/Pretty-Printing}

The extracted, pretty-printed, normalized filtered blocks are marked as potential clones using a Longest Common Subsequence (LCS) algorithm [@Hunt1977]. Then, a percentage of unique statements can be computed and, given the threshold $\alpha$, the blocks are marked as clones.

## Classifying Incoming Commits

The classification of incoming commits within CLEVER is a two-step process.
First, the commit goes through the statical part of the approach (Steps 1 to 4). 
If the statical part of the approach classifies the commit as _non-risky_, we simply let it through, and we stop the process.
If the commit is classified as _risky_, however, we continue the process with the Steps 5 to 9.
It is important to note that we do not output a _risky_ classification if we are unable to find a match in known defect-introducing signatures in our historical data. 

Finally, this two-step classification allows us to significantly reduce the time required, in average, to classify a commit.
If each commit has to be analysed against all known signatures in terms of code clone similarity, then, it would take more to process. From this perspective, the metric-based model can be seen as a first-level filter that finds suspicious commits.

## Proposing Fixes

Another important aspect of the design of CLEVER is the ability to provide guidance to developers on how to improve risky commits. We achieve this by extracting from the database the fix-commit corresponding to the matching defect-commit and present it to the developer. We believe that this makes CLEVER a practical approach for the developers as they will know why a given modification has been reported as risky in terms of code; this is something that is not supported by techniques based on statistical models (e.g., [@Kamei2013a; @Rosen2015]).  

Finally, using the fix used to fix past defects, we can provide a solution, in the form of a contextualised diff, to the developer that would remove the fault. A contextualised diff is a diff that is tempered with to match the current workspace of the developer regarding variable types and names.
In step 8, we adapt the matching fixes to the actual context of the developer by modifying indentation depth and variable name in an effort to reduce context switching [@Code; @Danny1997; @Altmann2004]. 
The contextualised fix is the fix to a matching buggy commit that we transform in terms of variable names and indentation.
We believe that it makes it easier for the developer to understand the proposed fix and see if it applies in their situation.
In table \ref{tab:bubble-fix} we present a fix transformation using the bubble sort example.

# Case Study Setup {#sec:exp}

In this section, we present the setup of our case study in terms of repository selection, dependency analysis, comparison process and evaluation measures.

## Project Repository Selection {#sec:rep}

In collaboration with Ubisoft developers, we selected 12 major software systems developed at Ubisoft to evaluate the effectiveness of CLEVER. These systems continue to be actively maintained by thousands of developers. Ubisoft projects are organized by game engines. A game engine can be used in the development of many high-budget games. The projects selected for this case study are related to the same game engine. For confidentiality and security reasons, neither the name nor the characteristics of these projects are provided. We can however disclose that the size of these systems altogether consists of millions of lines of code. 

## Project Dependency Analysis {#sec:dependencies}

Figure \ref{fig:dep-graph} shows the project dependency graph. 
As shown in Figure \ref{fig:dep-graph}, these projects are interconnected. 
A review of each cluster shows that this partitioning divides projects in terms of their high-level functionalities. For example, one cluster is related to a particular given family of video games whereas the other can be another family.
This project partitioning has been successfully validated by 11 software developers at Ubisoft.

## Building a Database of Defect-Commits and Fix-Commits {#sub:golden}

To build the database that we can use to assess the performance of CLEVER, we use the same process as discussed in Section \ref{sec:offline}.
We retrieve the full history of each project and label commits as defect-commits if they appear to be linked to a closed issue using the SZZ algorithm [@Kim2006c]. This baseline is used to compute the precision and recall of CLEVER. Each time CLEVER classifies a commit as _risky_; we can check if the _risky_ commit is in the database of defect-introducing commits. The same evaluation process is used by related studies  [@ElEmam2001; @Lee2011a; @Bhattacharya2011; @Kpodjedo2010;@Kamei2013].


## Process of Comparing New Commits {#sec:newcommits}

Because our approach relies on commit pre-hooks to detect risky commits, we had to find a way to *replay* past commits. 
To do so, we *cloned* our test subjects, and then created a new branch called *CLEVER*. When created, this branch is reinitialized at the initial state of the project (the first commit), and each commit can be replayed as they have originally been.  For each commit, we store the time taken for *CLEVER* to run, the number of detected clone pairs, and the commits that match the current commit.  As an example, suppose that we have three commits from two projects. 
At time $t_1$, commit $c_1$ in project $p_1$ introduces a defect. 
The defect is experienced by a user that reports it via an issue $i_1$ at $t_2$.
A developer fixes the defect introduced by $c_1$ in commit $c_2$ and closes $i_1$ at $t_3$.
From $t_3$ we known that $c_1$ introduced a defect using the process described in Section \ref{sub:golden}.
If at $t_4$, $c_3$ is pushed to $p_2$ and $c_3$ matches $c_1$ after preprocessing, pretty-printing and formatting, then $c_3$ is classified as _risky_ by CLEVER and $c_2$ is proposed to the developer as a potential solution for the defect introduced in $c_3$.

## Evaluation Measures

Similar to prior work focusing on risky commits (e.g., [@SunghunKim2008; @Kamei2013]), we used precision, recall, and F$_1$-measure to evaluate our approach. They are computed using TP (true positives), FP (false positives), FN (false negatives), which are defined as follows:


- TP:  is the number of defect-commits that were properly classified by CLEVERT
- FP:  is the number of healthy commits that were classified by CLEVERT as risky
- FN:  is the number of defect introducing-commits that were not detected by CLEVERT
- Precision: TP / (TP + FP)
- Recall: TP / (TP + FN)
- F$_1$-measure: 2.(precision.recall)/(precision+recall)

It is worth mentioning that, in the case of defect prevention, false positives can be hard to identify as the defects could be in the code but not yet reported through a bug report (or issue). To address this, we did not include the last six months of history. Following similar studies [@Rosen2015; @Chen2014; @Shivaji2013; @Kamei2013b], if a defect is not reported within six months then it is not considered.

# Case Study Results {#sec:result}

In this section, we show the effectiveness of CLEVER in detecting risky commits using a combination of metric-based models and clone detection. The main research question addressed by this case study is: _Can we detect risky commits by combining metrics and code comparison within and across related projects, and if so, what would be the accuracy?_

The experiments took nearly two months using a cluster of six 12 3.6 Ghz cores with 32GB of RAM. The most time consuming part of the experiment consists of building the baseline as each commit must be analysed with the SZZ algorithm. Once the baseline was established, the model built, it took, on average, 3.75 seconds to analyse an incoming commit on our cluster.

In the following subsections, we provide insights on the performance of CLEVER by comparing it to Commit-guru [@Rosen2015] alone, i.e., an approach that relies only on metric-based models. We chose Commit-guru because it has been shown to outperform other techniques (e.g., [@Kamei2013a; @Kpodjedo2010]). Commit-guru is also open source and easy to use. In addition, we compare CLEVER to itself when applied to all the projects, i.e., without applying the clustering step. This is important in order to validate the usefulness of the clustering step.

## Performance of CLEVER

CLEVER detects risky commits with a 79.10% precision, a 65.61% recall and a 71.72% F1 measure in average on the 12 projects we analysed at Ubisoft so far while using the threshold displayed on \ref{fig:CLEVERT2}. Any commit satisfying the two-steps classification but have a less than 30\% will be classified as _non-risky_. 
When applied to the same projects, Commit-guru achieves an average precision, recall and F1 measure of 66.71%, 63.01% and 64.80%, respectively.
The differences in performances between Commit-guru and CLEVER come from the fact that CLEVER is a two-steps classification where for a commit to be _risky_, a similar code has to be found and we use a different classification algorithm (random forest vs linear regression).
The second step of our classification allows us to drastically improve our accuracy (79.10% vs 66.71%) while achieving similar recall (65.61% vs. 63.01%).
Using Commit-guru as the baseline to measure the performances of CLEVER makes sense because we inspired ourselves from Commit-guru for the first step of our classification.
Moreover, Commit-guru performances have been repeatedly acknowledged by the community [@Rosen2015; @Kamei2013a; @misirli2016studying].

## Cluster Classifier Performances

Our clusters, computed with the dependencies of each project allow to solve one major problem in defect learning and prediction: cold start [@Schein2002].
Several approaches have already been proposed to solve these problems, however, they all require to classify _similar_ projects by hand and then, manipulate the feature space in order to adapt the model learnt from one project to the other [@Nam2013]. 
With our approach, the _similarity_ between projects is computed automatically and, results show that we do not actually need to manipulate the feature space.
Figures \ref{fig:bluecluster}, \ref{fig:yellowcluster} and \ref{fig:redcluster} show the performance of the CLEVER classification, in terms of ROC-curve and recall over precision curves, for the first thousand commit of the last project (chronologically) for the blue, yellow and red clusters presented in Figure \ref{fig:network-sample}.
The left sides or the graph are low cutoff (aggressive) while the right sides are high cutoff (conservative).
The area under the ROC curves are 0.817, 0.763 and 0.806 for the blue, yellow and red clusters, respectively.


\input{tex/clusterRoc}

\input{tex/workshop.tex}

These results show that not only the clusters are effective in identifying similar projects for defect learning transfer but also provide excellent performance while starting a new project. As new projects mature, their defects would be integrated into the model in order to improve it. To confirm this, we ran an experiment with the blue cluster where we first apply the model learnt from other members of the cluster for the first thousand commits.
The performance of this model is 75.1% precision, 57.6% recall for the first thousand commits.
After the first thousand commits, we take the commits of the day (for each day until the end of the project) and rebuild the model by adding the commits of the day to the ones already known from the clusters.
Overall, combining the commits from the project at-hand in a nightly fashion with the commits known from the cluster allowed us to correctly classified an additional 2.05\% of commits in the last 30\% of the project life compared to only using the commits from the project. 
This shows that, in addition to providing a viable alternative to a cold-start, our clusters also allow us to enhance the performances of the model at all time and no only at the start.

## Analysis of the Quality of the Fixes Proposed by CLEVER

In order to validate the quality of the fixes proposed by CLEVER, we conducted an internal workshop. 
In this workshop, participants were asked to assess the quality of proposed fixed when presented with the original buggy commit, the origin fix for this commit and the proposed one.
The attendance was composed of two software architects, two programmers, one technical lead and one IT project manager.

Our panel was asked to collectively review 12 randomly selected fixes coming from one given system. 
All the participants are familiar with the system.
The participants reviewed the fixes over a 70 minutes period, and we report their opinions in this section.
In addition to assessing the quality of the proposed fixes and their likeliness of the proposed fixes with actual fixes; we asked two additional questions to our pannel of experts: _Will you use CLEVER in the future?_ and _What aspects of CLEVER need to be improved?_. 

Table \ref{tab:Workshop} reports the results of our panel regarding accepted fix (\checkmark), fix refused ($x$) and non-applicable or unsure ($-$). 41.6% of the proposed bug fixes have been unanimously accepted  (1, 3, 6, 10 and 12) while 25% have been accepted by at least one member (4, 8, 11). 
We detail the refusals (x) and unsure (-) in the following.

$BF2$ was rejected by our panel as the region of the commit that triggered a match is, in fact, generated code. 
While this generated code is pushed into the repositories and can be part of bug fixing commit, the root cause of the bug lies in the code generator itself. 
Our proposed fix was proposing to update the generated code. 
While a match in the generated code could lead the developer to the root cause (i.e. the code generator) the question we ask our reviewers was _"Is the proposed fix applicable in the given situation?"_. 
In this occurrence, the proposed fix was not applicable.

$BF4$ was accepted by two reviewers and marked as unsure ($-$) by the rest of our panel.
Here, the reservation for the panelists that were unsure came from the lack of context surrounding the proposed fix.
Indeed, they were unable to determine if the fix was applicable or not without knowing what the original intent of the buggy commit was. 
In our reviewing session, we only provided the reviewers with the regions of the commits that match rather than the full commit.
Full commits can be lengthy as they can contain assets descriptions and generated code in addition to the actual code.
In this occurrence, the full context of the commit might have helped our reviewers to decide if the $BF4$ was applicable or not.
$BF5$ and $BF7$ were classified as unsure by the totality of our panel for the same reasons as $BF4$.
$BF8$ was refused by four of our reviewers and rejected by two is at felt to a reviewer that the code clone match was more a refactoring opportunity than a fix.
$BF12$ was marked as unsure by our panel because the code was corresponding to a subsystem that is maintained by another team and our panel judged itself unqualified.

For the two remaining questions we ask our reviewers  _Will you use MISFIRE in the future?_ and _What aspects need to be improved?_, they answered positively for the first question if some aspects are improved.
First, the reviewers expressed concerns about the context surrounding the buggy commit and the fixes.
While displaying the entire commits is not the solution, according to the panel, some context might be inferred from the commit message (which were not provided) and from the JIRA issue associated with the fix.
The second concern of our panel, which generated many discussions, is the incapacity of CLEVERT to match generated code.
In the occurrence of a match in generated code, CLEVERT should be able to identify it and point towards the code generator rather than the generated code.

One interesting remark made by the panel is that were expected buggy commit, and their associated proposed fix to be more complex (i.e. network, threading, physics) than the one we presented.
Ultimately, they agreed that the bug showcased were rather simple and could have been caught during a code review by an experienced developer.

# Discussion {#sec:threats}

In this section, we propose a discussion on limitations and threats to validity.

## Limitations

We identified two main limitations of our approach, CLEVER, which require further studies.

CLEVERT is designed to work on multiple related systems. Applying CLEVER on a single system will most likely be less effective as the the two steps classification would be hindered by the fact that it is unlikely to have a large number of similar bugs within the same system. For single systems, we recommend the use of statistical models based on process and code metrics for the detection of risky commits such as the ones developed by Kamei et al. and Rosen et al. [@Kamei2013; @Rosen2015]. A metric-based solution, however, may turn to be ineffective when applied across systems because of the difficulty associated with identifying common thresholds that are applicable to a wide range of systems.    

The second limitation we identified has to do with the fact that CLEVER is designed to work with Ubisoft systems. Ubisoft uses C\#, C, C++, Java and other internally developped languages. It is however common to have  other languages used in an environment with many inter-related systems. We intend to extend CLEVERT to process commits from other languages as well.

## Threats to Validity

The selection of target systems is one of the common threats to validity for approaches aiming to improve the analysis of software systems. It is possible that the selected programs share common properties that we are not aware of and therefore, invalidate our results. However, the systems vary in terms of purpose, size, and history. 

In addition, we see a threat to validity that stems from the fact that we only used closed-source systems. 
The results may not be generalizable to open-source systems.

The programs we used in this study are all based on the C\#, C, C++ and Java programming language. 
This can limit the generalization of the results to projects written in other languages. 

Finally, part of the analysis of the CLEVER proposed fixes that we did was based on manual comparisons of the CLEVERT fixes with those proposed by developers with a focus group composed of experienced engineers and software architects. Although we exercised great care in analyzing all the fixes, we may have misunderstood some aspects of the commits.  

In conclusion, internal and external validity have both been minimized by choosing a set of 12 different systems, using input data that can be found in any programming languages and version systems (commit and changesets).

# Conclusion {#sec:conclusion}

In this paper, we presented CLEVER (Combining Levels of Bug Prevention and Resolution Techniques), an approach that detects risky commits (i.e., a commit that is likely to introduce a bug) with an average of 79.10% precision and a 65.61% recall.
CLEVERT combines code metrics, clone detection techniques and project dependency analysis to detect risky commits within and across projects.  CLEVERT operates at commit-time, i.e., before the commits reach the central code repository. Also, because it relies on code comparison, CLEVER does not only detect risky commits but also makes recommendations to developers on how to fix them. We believe that this makes CLEVER a practical approach for preventing bugs and proposing corrective measures that integrate well with the developer's workflow through the commit mechanism.  

As a future work, we want to build a feedback loop between the users and the clusters of known buggy commits and their fixes.
If a fix is never used by the end-users, then we could remove it from the clusters and improve our accuracy.

# Reproduction Package

For security and confidentiality reasons we cannot provide a reproduction package that will inevitably involve Ubisoft's copyrighted source code.
However, the CLEVER source code is in the process of being open-sourced and will be soon available at https://github.com/ubisoftinc.

# Acknowledgments

We are thankful to Olivier Pomarez, Yves Jacquier, Nicolas Fleury, Alain Bedel, Mark Besner, David Punset, Paul Vlasie, Cyrille Gauclin, Luc Bouchard, Chadi Lebbos and Florent Jousset, Anthony Brien, Thierry Jouin and Jean-Pierre Nolin from Ubisoft for their participations in validating CLEVERT hypothesis, efficiency and the fixes proposed by our approach.

\section*{References} 

<!-- Footnotes text -->

[^graphwalker-link-44]: https://github.com/jrtom/jung/issues/44
[^BatchEE-link-69]: https://issues.apache.org/jira/browse/BATCHEE-69
[^react-native]: https://github.com/facebook/react-native
[^maven]: https://maven.apache.org/
[^txl]:http://txl.ca


<!-- End Footnotes text -->

\small
\setlength{\parindent}{0pt}
\setlength{\parskip}{6pt plus 2pt minus 1pt}

