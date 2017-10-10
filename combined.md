---
title: "MISFIRE: Combining Code Metrics with Clone Detection for Just-In-Time Fault Prevention and Resolution in Ultra-Large Multi-Projects Industrial Datasets"
bibliography: config/library.bib
abstract:  

author: 
- name: Mathieu Nayrolles
  affiliation: La Forge Research Lab, Ubisoft
  location: Montréal, QC, Canada
  email: mathieu.nayrolles@ubisoft.com
- name: Abdelwahab Hamou-Lhadj
  affiliation: SBA Lab, ECE Dept, Concordia University
  location: Montréal, QC, Canada
  email: wahab.hamou-lhadj@concordia.ca
- name: Emad Shihab
  affiliation: DAS Lab, CSE Dept, Concordia University
  location: Montréal, QC, Canada
  email: eshihab\@cse.concordia.ca
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

\IEEEPARstart{R}{esearch} in software maintenance has evolved over the year to include areas like mining bug repositories, bug analytic, and bug prevention and reproduction. The ultimate goal is to develop better techniques and tools to./ help software developers detect, correct, and prevent bugs in an effective and efficient manner. 

One particular (and growing) line of research focuses on the problem of preventing the introduction of bugs by detecting risky commits (preferably before the commits reach the central repository). Recent approaches (e.g., [@Lo2013; @Nam2013]) rely on training models based on code and process metrics (e.g., code complexity, experience of the developers, etc.) that are used to classify new commits as risky or not. 

Despite the recent advances in the field, the literature shows that many existing software maintenance tools have yet to be adopted by industry [@Lewis2013; @Foss2015; @Layman2007; @Ayewah2007; @Ayewah2008; @Johnson2013; @Norman2013; @Hovemeyer2004; @Lopez2011]. After assessing existing commercially available tools and state of the art approaches proposed in the litterature with software architects, team leaders, and senior developers, we extracted factors that we believe are contributing to this situation:

- Integration with the developer's workflow: Most existing maintenance tools ([@Kim2006a; @Ayewah2008b; @Findbugs2015; @Moha2010; @Palma; @Nayrolles2013d; @Nayrolles; @Nayrolles2013a;  @Nayrolles2015a] are some noticeable examples) are not integrated well with the work flow of software developers (i.e., coding, testing, debugging, committing). Using these tools, developers have to download, install and understand them to achieve a given task. They would constantly need to switch from one workspace to another for different tasks (i.e., feature location with a command line tool, development and testing  code with an IDE, development and testing front end code with another IDE and a browser, etc.) [@Robertson2004; @Robertson2006; @Beckwith2006].

- Corrective actions: The outcome of these tools does not always lead to corrective actions that the developers can implement. Most of these tools return several results that are often difficult to interpret by developers. Take for example, FindBugs [@Hovemeyer2004], a popular bug detection tool. This tool detects hundreds of bug signatures and reports them using an abbreviated code such as `CO_COMPARETO_INCORRECT_FLOATING`. Using this code, developers can browse the FindBug's dictionary and find the corresponding definition _This method compares double or float values using pattern like this: `val1 > val2 ? 1 : val1 < val2 ? -1 : 0`_. While the detection of this bug pattern is accurate, the tool does not propose any corrective actions to the developers that can help them fix the problem. Moreover, it has been reported in the literature that the output of existing maintenance tools tends to be verbose at the point where developers decide to simply ignore them [@Arai2014; @Kim2007b; @Kim2007c; @Ayewah2010; @Shen2011].

- Leverage of historical data: These tools do not leverage a large body of knowledge that already exists in open source or within the organization.  For defect prevention, for example, the state of the art approaches consists of adapting statistical models built for one project to another project [@Lo2013; @Nam2013].	As argued by Lewis _et al._ [@Lewis2013] and Johnson _et al._ [@Johnson2013], approaches based solely on statistical models are perceived by developers as black box solutions. Developers are less likely to trust the output of these tools.

In this paper, we propose an approach that adresses these challenges whithin the framework of ultra-large repositories at Ubisoft.

Ubisoft, one of world's largest video game development companies, specializes in the design and implementation of high-budget video games. 
Such high-budget video games involve thousands of highly qualified personels from various fields (developers, artists, management, marketing, ...).
To give the reader a grasp over the actual size of such software projects, the final product can contain millions of files and hundreds of thousands of commits.   
Furthermore, several high-budget games are under development at the same time by 8,000 developers scatered accross the locations on six continents. 

Our approach named MISFIRE (MetrIcS and clone detection for Fault Interception and Removing in ultra-large rEpositories) leverages three decades of code history in a cross-projects manner with the aims to automatically detecte and propose faults correction before they reach the central software repository. 
More specifically, it uses a combination of well-known code metrics to build statistical models in order to prevent faults insertion. In addition to these metrics, we also use an in-house clone detector that extracts code blocks from incoming commits and compare them to those of known defect-introducing commits. Finally, using the fix used to fix past defects, we are able to provide a solution, in the form of a contextualized diff, to the developer that would remove the fault. A contextualized diff is a diff that is tempered with to match the current workspace of the developer in terms of variable types and names.

This novel approach performs in the same ranges as state-of-the-art approaches such as Code Guru when it comes to defect introduction detection. Indeed, we are able to detect defects introduction with a 71% precision and a 70% recall.
In addition we ask in-house experts to manually evaluat the quality of the fix we propose to remove the faults from the current code submission. 
More than 62% of the analyzed proposed fixes were judged highly relevant and another 12% were classified as relevant. 
Overall, 74% of the proposed fixes help developers toward the craft of an actual fix.
Finally, MISFIRE addresses the three factors we identified as contributing to the slow adotpion of automatic defect prevention tools in large companies (Integration with the developer's workflow, Corrective actions and Leverage of historical data).

The remaining parts of this paper are organized as follows. In Section \ref{sec:relwork}, we present related work. Sections \ref{sec:misfire},  \ref{sec:exp} and \ref{sec:result} are dedicated to the misfire approach, the case study setup, and the case study results.
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

Our work shares a similar goal to the work on the prediction of risky changes, however, misfire takes a different approach in that it leverages dependencies of a project to determine risky changes.


## Transfert Defect Learning

## Automatic Patch Generation

Since misfire not only flags risky changes, but also provides developers with fixes that have been applied in the past, automatic patch generation work is also related. Pan *et al.* [@Pan2008] identified 27 bug fixing patterns that can be applied to fix software bugs in Java programs. They showed that between 45.7 - 63.6\% of the bugs can be fixed with their patterns. Later, Kim *et al.* [@Kim2013] generated patches from human-written patches and showed that their tool, PAR, successfully generated patches for 27 of 119 bugs. Tao *et al.* [@tao2014automatically] also showed that automatically generated patches can assist developers in debugging tasks. Other work also focused on determining how to best generate acceptable and high quality patches, e.g. [@Dallmeier; @le2012systematic], and determine what bugs are best fit for automatic patch generation [@le2015should].

Our work differs from the work on automated patch generation in that we do not generate patches, rather we use clone detection to determine the similarity of a change to a previous risky change and suggest to the developer the fixes of the prior risky changes.

# The MISFIRE Approach {#sec:misfire}

Figures \ref{fig:misfire1}, \ref{fig:misfire3} and \ref{fig:misfire2} show an overview of the misfire approach, which consists of two parallel processes. 

In the first process (Figures \ref{fig:misfire1} and \ref{fig:misfire3}), misfire manages events happening on project tracking systems to extract defect-introducing commits and commits that provided the fixes. For simplicity, in the rest of this paper, we refer to commits that are used to fix defects as _fix-commits_. 
We use the term _defect-commit_ to mean a commit that introduces a defect. In the second phase, misfire analyses the developer's new commits before they reach the central repository to detect potential risky commits (commits that may introduce bugs). 


\input{tex/approach}

The project tracking component of misfire listens to bug (or issue) closing events of major open-source projects (currently, misfire is tested with 42 large projects). These projects share many dependencies. Projects can depend on each other or common external tools and libraries. We perform project dependency analysis to identify groups of highly-coupled projects. 

In the second process (Figure \ref{fig:misfire2}), misfire identifies risky commits within each group to increase the chances of finding risky commits caused by project dependencies. 
For each project group, we extract code blocks from defect-commits and fix-commits.  
The extracted code blocks are saved in a database that is used to identify risky commits before they reach the central repository. 
For each match between a risky commit and a _defect-commit_, we pull out from the database the corresponding _fix-commit_ and present it to the developer as a potential way to improve the commit content. 
These phases are discussed in more detail in the upcoming subsections.

## Clustering Project Repositories {#sec:clustering}

We cluster projects according to their dependencies.  The rationale is that projects that share dependencies are most likely to contain defects caused by misuse of these dependencies. In this step, the project dependencies are analysed and saved into a single NoSQL graph database as shown in Figure \ref{fig:misfire3}. Graph databases use graph structures as a way to store and query information.  In our case,  a node corresponds to a project that is connected to other projects on which it depends. Project dependencies can be automatically retrieved if projects use a dependency manager such as Maven. 

Figure \ref{fig:network-sample} shows a simplified view of a dependency graph for a project named \texttt{com.badlogicgames.gdx}.
As we can see, \texttt{com.badlogicgames.gdx} depends on projects owned by the same organization (i.e., badlogicgames) and other organizations such as Google, Apple, and Github.

\input{tex/network-sample}

Once the project dependency graph is extracted, we use a clustering algorithm to partition the graph. To this end, we choose the Girvan–Newman algorithm [@Girvan2002; @Newman2004], used to detect communities by progressively removing edges from the original network. Instead of trying to construct a measure that identifies the edges that are the most central to communities, the Girvan–Newman algorithm focuses on edges that are most likely "between" communities. This algorithm is very effective at discovering community structure in both computer-generated and real-world network data [@Newman2004]. Other clustering algorithms can also be used. 

## Building a Database of Code Blocks of Defect-Commits and Fix-Commits {#sec:offline}

To build our database of code blocks that are related to defect-commits and fix-commits, we first need to identify the respective commits. Then, we extract the relevant blocks of code from the commits.

**Extracting Commits:** misfire listens to bug (or issue) closing events happening on the project tracking system. Every time an issue is closed, misfire retrieves the commit that was used to fix the issue (the fix-commit) as well as the one that introduced the defect (the defect-commit). Retrieving fix-commits, however, is known to be a challenging task [@Wu2011]. This is because the link between the project tracking system and the code version control system is not always explicit. In an ideal situation, developers would add a reference to the issue they work on inside the description of the commit. However, this good practice is not always followed. To make the link between fix-commits and their related issues, we turn to a modified version of the back-end of commit-guru [@Rosen2015]. Commit-guru is a tool, developed by Rosen _et al._  [@Rosen2015] to detect _risky commits_. In order to identify risky commits, Commit-guru builds a statistical model using change metrics (i.e.,  amount of lines added, amount of lines deleted, amount of files modified, etc.) from past commits known to have introduced defects in the past.

Commit-guru's back-end has three major components: ingestion, analysis, and prediction. We reuse the ingestion and the analysis part for misfire. The ingestion component is responsible for ingesting (i.e., downloading) a given repository.
Once the repository is entirely downloaded on a local server, each commit history is analysed. Commits are classified using the list of keywords proposed by Hindle *et al.* [@Hindle2008]. Commit-guru implements the SZZ algorithm [@Kim2006c] to detect risky changes, where it performs the SCM blame/annotate function on all the modified lines of code for their corresponding files on the fix-commit's parents. This returns the commits that previously modified these lines of code and are flagged as the defect introducing commits (i.e., the defect-commits). Prior work showed that commit-guru is effective in identifying defect-commits and their corresponding fixing commits [@Kamei2013a] and the SZZ algorithm, used by commit-guru, is shown to be effective in detecting risky commits [@Kamei2013; @Rosen2015]. Note that we could use a simpler and more established approach such as Relink [@Wu2011] to link the commits to their issues and re-implement the classification proposed by Hindle *et al.* [@Hindle2008] on top of it. However, commit-guru has the advantage of being open-source, making it possible to modify it to fit our needs by fine-tuning its performance.

**Extracting Code Blocks:** To extract code blocks from fix-commits and defect-commits,  we rely on TXL [@Cordy2006a], which is a first-order functional programming over linear term rewriting, developed by Cordy et al. [@Cordy2006a]. For TXL to work, one has to write a grammar describing the syntax of the source language and the transformations needed. TXL has three main phases: *parse*, *transform*, *unparse*. In the parse phase, the grammar controls not only the input but also the output forms. The following code sample---extracted from the official documentation---shows a grammar matching an *if-then-else* statement in C with some special keywords: [IN] (indent), [EX] (exdent) and [NL] (newline) that will be used in the output form.

```c
define if_statement
  if ( [expr] ) [IN][NL]
[statement] [EX]
[opt else_statement]
end define

define else_statement
  else [IN][NL]
[statement] [EX]
end define
```

Then, the *transform* phase applies transformation rules that can, for example, normalize or abstract the source code. 
Finally, the third phase of TXL, called *unparse*, unparses the transformed parsed input to output it. 
Also, TXL supports what its creators call _Agile Parsing_ [@Dean], which allow developers to redefine the rules of the grammar and, therefore, apply different rules than the original ones.
misfire takes advantage of that by redefining the blocks that should be extracted for the purpose of code comparison, leaving out the blocks that are out of scope. More precisely, before each commit, we only extract the blocks belonging to the modified parts of the source code.  Hence, we only process, in an incremental manner, the latest modification of the source code instead of the source code as a whole.

We have selected TXL for several reasons. First, TXL is easy to install and to integrate with the normal workflow of a developer. 
Second, it was relatively easy to create a grammar that accepts commits as input. This is because TXL supports C, Java, Csharp, Python and WSDL grammars, with the ability to customize them to accept changesets (chunks of the modified source code that include the added, modified, and deleted lines) instead of the whole code.

\input{tex/extract}

Algorithm \ref{alg:extract} presents an overview of the *extract* and *save* blocks operations of misfire. This algorithm receives as argument, the changesets and the blocks that have been previously extracted. 
Then, Lines 1 to 5 show the $for$ loop that iterates over the changesets. 
For each changeset (Line 2), we extract the blocks by calling the $~extract\_blocks(Changeset~cs)$ function. 
In this function, we expand our changeset to the left and to the right in order to have a complete block.

As depicted below, changesets contain only the modified chunk of code and not necessarily complete blocks. 

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
We do so by checking the block's beginning and ending with a parentheses algorithms [@bultena1998eades]. 
Then, we send these expanded changesets to TXL for block extraction and formalization.

One important note about this database is that the process can be cold-started.
A tool supporting misfire does not need to *wait* for a project to have issues and fixes to be in effect.
It can leverage the defect-commits and fix-commits
of projects in the same cluster that already have a history.
Therefore, misfire is applicable at the beginning of every
project. The only requirement is to use a dependency manager. 

## Analysing New Commits Using Pre-Commit Hooks {#sec:online}

Each time a developer makes a commit, misfire intercepts it using a pre-commit hook,  extracts the corresponding code block (in a similar way as in the previous phase), and compares it to the code blocks of historical defect-commits. If there is a match then the new commit is deemed to be risky. A threshold $\alpha$ is used to assess the extent beyond which two commits are considered similar. The setting of $\alpha$ is discussed in the case study section.

Pre-commit hooks are custom scripts set to fire off when certain important actions of the versioning process occur.
There are two groups of hooks: client-side and server-side. Client-side hooks are triggered by operations such as committing and merging, whereas server-side hooks run on network operations such as receiving pushed commits. These hooks can be used for all sorts of reasons such as checking compliance with coding rules or automatic runs of unit test suites. The pre-commit hook runs before the developer specifies a commit message. It is used to inspect the modifications that are about to be committed. misfire is based on a set of bash and python scripts, and the entry point of these scripts lies in a pre-commit hook. These scripts intercept the commit and extract the corresponding code blocks.

To compare the extracted blocks to the ones in the database, we resort to clone detection techniques, more specifically, text-based clone detection techniques. This is because lexical and syntactic analysis approaches (alternatives to text-based comparisons) would require a complete program to work, i.e., a program that compiles.  In the relatively wide-range of tools and techniques that exist to detect clones by considering code as text [@Johnson1993;  @Johnson1994; @Marcus; @Manber1994; @StephaneDucasse; @Wettel2005], we selected NICAD as the main text-based method for comparing code blocks [@Cordy2011] for several reasons. 
First, NICAD is built on top of TXL, which we also used in the previous phase.  Second, NICAD can detect Types 1, 2 and 3 software clones [@CoryKapser]. Type 1 clones are copy-pasted blocks of code that only differ from each other in terms of non-code artefacts such as indentation, whitespaces, comments and so on.  Type 2 clones are blocks of code that are syntactically identical except literals, identifiers, and types that can be modified. Also, Type 2 clones share the particularities of Type 1 about indentation, whitespaces, and comments. Type 3 clones are similar to Type 2 clones in terms of modification of literals, identifiers, types, indentation, whitespaces, and comments but also contain added or deleted code statements. misfire detects Type 3 clones since they can contain added or deleted code statements, which make them suitable for comparing commit code blocks.
The problem with the current implementation of NICAD is that it only considers complete Java, C\#, and C files. We needed to adapt NICAD to process changesets. With the aid of TXL designers (through their online forum), we developed a TXL grammar for changesets.

NICAD works in three phases: *Extraction*, *Comparison* and *Reporting*. 
During the *Extraction* phase all potential clones are identified, pretty-printed, and extracted. 
We do not use the *Extraction* phase of NICAD as it has been built to work on programs that are syntactically correct, which is not the case for changesets. 
We replaced NICAD’s *Extraction* phase with our scripts for building code blocks (described in the previous phase).

In the *Comparison* phase, the extracted blocks are transformed, clustered and compared to find potential clones. 
Using TXL sub-programs, blocks go through a process called pretty-printing where they are stripped of formatting and comments. 
When code fragments are cloned, some comments, indentation or spacing are changed according to the new context where the new code is used. 
This pretty-printing process ensures that all code will have the same spacing and formatting, which renders the comparison of code fragments easier. 
Furthermore, in the pretty-printing process, statements can be broken down into several lines. Table \ref{tab:pretty-printing} [@Iss2009] shows how this can improve the accuracy of clone detection with three `for` statements:

\begin{equation}
 for (i=0; i<10; i++)
\end{equation}

\begin{equation}
 for (i=1; i<10; i++)
\end{equation}

\begin{equation}
 for (j=2; j<100; j++)
\end{equation}

The pretty-printing allows NICAD to detect Segments 1 and 2 as a clone pair because only the initialization of $i$ changed. This specific example would not have been marked as a clone by other tools we tested such as Duploc [@StephaneDucasse]. 
In addition to the pretty-printing, code can be normalized and filtered to detect different classes of clones and match user preferences.

\input{tex/Pretty-Printing}

The extracted, pretty-printed, normalized and filtered blocks are marked as potential clones using a Longest Common Subsequence (LCS) algorithm [@Hunt1977]. Then, a percentage of unique statements can be computed and, given the threshold $\alpha$, the blocks are marked as clones.

Another important aspect of the design of misfire is the ability to provide guidance to developers on how to improve risky commits. We achieve this by extracting from the database the fix-commit corresponding to the matching defect-commit and present it to the developer. We believe that this makes misfire a practical approach for the developers as they will know why a given modification has been reported as risky in terms of code; this is something that is not supported by techniques based on statistical models (e.g., [@Kamei2013a; @Rosen2015]).  
A tool that supports misfire should have enough flexibility to allow developers to enable or disable the recommendations made by misfire. 
Furthermore, because misfire acts before the commit reach the central repository, it prevents unfortunate pulls of defects by other members of the organization. 

# Case Study Setup {#sec:exp}

In this section, we present the setup of our case study in terms of repository selection, dependency analysis, comparison process and evaluation measures.

## Project Repository Selection {#sec:rep}

To select the projects used to evaluate our approach, we followed three simple criteria. First, the projects need to be in Java and use Maven to manage dependencies. This way, we can automatically extract the dependencies and perform the clustering of projects. The second criterion is to have projects that enjoy a large community support and interest. We selected projects that have at least 2000 followers. Finally, the projects must have a public issue repository to be able to mine their past issues and the fixes. We queried Github with these criteria and retrieved 42 projects (see Table \ref{tab:results} for the list of projects), including those from some of major open-source contributors such as Alibaba, Apache Software Foundation, Eclipse, Facebook, Google and Square. 

## Project Dependency Analysis {#sec:dependencies}

Figure \ref{fig:dep-graph} shows the project dependency graph. The dependency graph is composed of 592 nodes divided into five clusters shown in yellow, red, green, purple and blue. The size of the nodes in Figure \ref{fig:dep-graph} is proportional to the number of connections from and to the other nodes.

\input{tex/dependencies}

As shown in Figure \ref{fig:dep-graph},  these Github projects are very much interconnected. On average, the projects composing our dataset have 77 dependencies. Among the 77 dependencies, on average, 62 dependencies are shared with at least one other project from our dataset.

Table \ref{tab:communities} shows the result of the Girvan–Newman clustering algorithm  in terms of centroids and betweenness. The blue cluster is dominated by Storm from The Apache Software Foundation. Storm is a distributed real-time computation system. Druid by Alibaba, the e-commerce company that provides consumer-to-consumer, business-to-consumer and business-to-business sales services via web portals, dominates the yellow cluster. In recent years, Alibaba has become an active member of the open-source community by making some of its projects publicly available. The red cluster has Hadoop by the Apache Software Foundation as its centroid. Hadoop is an open-source software framework for distributed storage and distributed processing of very large datasets on computer clusters built from commodity hardware. The green cluster is dominated by the Persistence project of OpenHab.  OpenHab proposes home automation solutions and the Persistence project is their data access layer. Finally, the purple cluster is dominated by Libdx by Badlogicgames, which is a cross-platform framework for game development.

A review of each cluster shows that this partitioning divides projects in terms of high-level functionalities. For example, the blue cluster is almost entirely composed of projects from the Apache Software Foundation. Projects from the Apache Software Foundation tend to build on top of one another. We also have the red cluster for Hadoop, which is by itself an ecosystem inside the Apache Software Foundation. Finally, we obtained a cluster for e-commerce applications (yellow), real-time network application for home automation (green), and game development (purple).


\input{tex/clusters}


## Building a Database of Defect-Commits and Fix-Commits for Performances Evaluation {#sub:golden}

To build the database that we can use to assess the performance of misfire, we use the same process as discussed in Section \ref{sec:offline}. We used Commit-guru to retrieve the complete history of each project and label commits as defect-commits if they appear to be linked to a closed issue. The process used by Commit-guru to identify commits that introduce a defect is simple and reliable in terms of accuracy and computation time [@Kamei2013]. We use the commit-guru labels as the baseline to compute the precision and recall of misfire. Each time misfire classifies a commit as _risky_, we can check if the _risky_ commit is in the database of defect-introducing commits. The same evaluation process is used by related studies  [@ElEmam2001; @Lee2011a; @Bhattacharya2011; @Kpodjedo2010].

## Process of Comparing New Commits {#sec:newcommits}

Because our approach relies on commit pre-hooks to detect risky commits, we had to find a way to *replay* past commits. 
To do so, we *cloned* our test subjects, and then created a new branch called *misfire*. When created, this branch is reinitialized at the initial state of the project (the first commit) and each commit can be replayed as they have originally been.  For each commit, we store the time taken for *misfire* to run, the number of detected clone pairs, and the commits that match the current commit. 
As an example, let's assume that we have three commits from two projects. 
At time $t_1$,  commit $c_1$ in project $p_1$ introduces a defect. 
The defect is experienced by an user that reports it via an issue $i_1$ at $t_2$.
A developer fixes the defect introduced by $c_1$ in commit $c_2$ and closes $i_1$ at $t_3$.
From $t_3$ we known that $c_1$ introduced a defect using the process described in Section \ref{sub:golden}.
If at $t_4$, $c_3$ is pushed to $p_2$ and $c_3$ matches $c_1$ after preprocessing, pretty-printing and formatting, then $c_3$ is classified as _risky_ by misfire and $c_2$ is proposed to the developer as a potential solution for the defect introduced in $c_3$.

\input{tex/alpha}

To measure the similarity between pairs of commits, we need to decide on the value of $\alpha$. One possibility would be to test for all possible values of $\alpha$ and pick the one that provides best accuracy (F$_1$-measure). The ROC (Receiver Operating Characteristic) curve can then be used to display the performance of misfire with different values of $\alpha$. Running experiments with all possible $\alpha$ turned out to be computationally demanding given the large number of commits. Testing with all the different values of $\alpha$ amounts to 4e10 comparisons. 

To address this, we randomly selected a sample of 1700 commits from our dataset and checked the results by varying $\alpha$ from 1 to 100%. Figure \ref{fig:alpha-deter} shows the results. The best trade-off between precision and recall is obtained when $\alpha$ = 35\%. This threshold is in line with the findings of Roy et al. [@Roy2008; @Cordy2011] who showed through empirical studies that using NICAD with a threshold of around 30\%, the default setting, provides good results for the detection of Type 3 clones. For these reasons, we set $\alpha$ = 35\% in our experiments.

## Evaluation Measures

Similar to prior work focusing on risky commits (e.g., [@SunghunKim2008; @Kamei2013]), we used precision, recall, and F$_1$-measure to evaluate our approach. They are computed using TP (true positives), FP (false positives), FN (false negatives), which are defined as follows:


- TP:  is the number of defect-commits that were properly classified by misfire
- FP:  is the number of healthy commits that were classified by misfire as risky
- FN:  is the number of defect introducing-commits that were not detected by misfire
- Precision: TP / (TP + FP)
- Recall: TP / (TP + FN)
- F$_1$-measure: 2.(precision.recall)/(precision+recall)

It is worth mentioning that, in the case of defect prevention, false positives can be hard to identify as the defects could be in the code but not yet reported through a bug report (or issue). To address this, we did not include the last six months of history. Following similar studies [@Rosen2015; @Chen2014; @Shivaji2013; @Kamei2013b], if a defect is not reported within six months then it is not considered.

# Case Study Results {#sec:result}

In this section, we show the effectiveness of misfire in detecting risky commits using clone detection and project dependency analysis. The main research question addressed by this case study is: _Can we detect risky commits using code comparison within and across related projects, and if so, what would be the accuracy?_

Table \ref{tab:results} shows the results of applying misfire in terms of the organization, project name, a short description of the project, the number of classes, the number of commits, the number of defect-commits, the number of defect-commits detected by misfire, precision (%), recall (%), F$_1$-measure and the average difference, in days, between detected commit and the _original_ commit inserting the defect for the first time. 

With $\alpha$ = 35%, misfire achieves, on average, a precision of 90.75% (13,899/15,316) commits identified as risky. These commits triggered the opening of an issue and had to be fixed later on. On the other hand, misfire achieves, on average, 37.15% recall (15,316/41,225), and an average F$_1$ measure of 52.72%. 

\input{tex/result}		

The relatively _low_ recall is to be expected, since misfire classifies commits as risky only if a similar defect-introducing commit happened in one of the 42 open-source projects. 

Also, out of the 15,316 commits misfire classified as _risky_, only 1,320 (8.6%) were because they were matching a defect-commit inside the same project.
This finding supports the idea that developers of a project are not likely to introduce the same defect twice while developers of different projects that share dependencies are, in fact, likely to introduce similar defects. We believe this is an important finding for researchers aiming to achieve cross-project defect prevention, regardless of the technique (e.g., statistical model, AST comparison, code comparison, etc.) employed.

It is important to note that we do not claim that 37.15% of issues in open-source systems are caused by project dependencies. 
To support such a claim, we would need to analyse the 15,316 detected defect-commits and determine how many yield defects that are similar across projects. 

Studying the similarity of defects across projects is a complex task and may require analysing the defect reports manually. This is left as  future work. 
That said, we showed, in this paper, that software systems sharing dependencies also share common issues, irrespective of whether these issues represent similar defects or not.

The experiments took nearly three months using 48 Amazon Virtual Private Servers running in parallel. When deployed, the most time consuming part of misfire was spent on building the model of known bug-introducing commits. Once the model was built, it took, on average, 72 seconds to analyze an incoming commit on a typical workstation (quad-core @ 3.2GHz with 8 GB of RAM). 

In the following subsections, we compare misfire with a random classifier, analyze the best and worst performing projects and assess the quality of the proposed fixes.


## Baseline Classifier Comparison

Although our average F$_1$ measure of 52.72% may seem low at first glance, achieving a high F$_1$ measure for unbalanced data is very difficult [@menzies2007problems]. Therefore, a common approach to ground detection results is to compare it to a simple baseline.

To the best of our knowledge, this is the first approach that relies on  code similarity instead of code or process metrics for the detection of risky commits. Comparing it to other approaches will not be accurate. In addition, existing metric-based techniques (e.g., [@Nam2013]) detect risky commits within single projects only. misfire, on the other hand, operates across projects. We compared misfire  with a random classifier to have a baseline and show that we perform better than a simple baseline.

The baseline classifier first generates a random number $n$ between 0 and 1 for the 165,912 commits composing our dataset.
For each commit, if $n$ is greater than 0.5, then the commit is classified as risky and vice versa. As expected by a random classifier, our implementation detected ~50% (82,384 commits) of the commits to be _risky_. It is worth mentioning that the random classifier achieved 24.9% precision, 49.96% recall and 33.24% F$_1$-measure. Since our data is unbalanced (i.e., there are many more _healthy_ than _risky_ commits) these numbers are to be expected for a random classifier. Indeed, the recall is very close to 50% since a commit can take on one of two classifications, risky or non-risky. While analysing the precision, however, we can see that the data is unbalanced (a random classifier would achieve a precision of 50% on a balanced dataset).

It is important to note that the purpose of this analysis is not to say that we outperform a simple random classifier, rather to shed light on the fact that our dataset is unbalanced and achieving an average F$_1$ = 52.72% is non-trivial, especially when a baseline only achieves an F$_1$-measure of 33.24%.

## Performance of misfire 

In this section, we provide insight on the performance of misfire by examining the projects for which the best and worst results were obtained.

misfire performed best when applied to three projects: Otto by Square (100.00% precision and 76.61% recall, 96.55% F$_1$-measure), JStorm by Alibaba (90.48% precision, 87.50% recall, 88.96% F$_1$-measure), and Auto by Google (90.48% precision, 87.50% recall, 86.76% F$_1$-measure). It performed worst when applied to Android Annotations by Excilys (100.00% precision, 1.59% recall, 3.13% F$_1$-measure) and Che by Eclipse (88.89% precision, 5.33% recall, 10.05% F$_1$-measure), Openhab by Openhab (100.00% precision, 7.14% recall, 13.33% F$_1$-measure). To understand the performance of misfire, we conducted a manual analysis of the commits classified as _risky_ by misfire for these projects.

### Otto by Square (F$_1$-measure = 96.5%)

At first, the F$_1$-measure of Otto by Square seems surprising given the specific set of features it provides. Otto provides a Guava-based event bus.  While it does have dependencies that make it vulnerable to defects in related projects, the fact that it provides specific features makes it, at first sight, unlikely to share defects with other projects.
Through our manual analysis, we found that out of the 16 _risky_ commits detected by misfire, 11 (68.75%) matched defect-introducing commits inside the Otto project itself. This is significantly higher than the average number of single-project defects (8.6%). Further investigation of the project management system revealed that very few issues have been submitted for this project (15) and, out of the 11 matches inside the Otto project, 7 were aiming to fix the same issue that had been submitted and fixed several times instead of re-opening the original issue.

### JStorm by Alibaba (F$_1$-measure = 88.96%)

For JStorm by Alibaba, our manual analysis of the _risky_ commits revealed that, in addition to providing stream processes, JStorm mainly supports JSON. The commits detected as _risky_ were related to the JSON encoding/decoding functionalities of JStorm.  In our dataset, we have several other projects that supports JSON encoding and decoding such as FastJSON by Alibaba, Hadoop by Apache, Dropwizard by Dropwizard, Gradle by Gradle and Anthelion by Yahoo.
There is, however, only one project supporting JSON in the same cluster as JStorm, Fastjson by Alibaba.
FastJSON has a rather large history of defect-commits (516) and 18 out of the 21 commits marked as _risky_ by misfire were marked so because they matched defect-commits in the FastJSON project.

### Auto by Google (F$_1$-measure = 86.76%)

Google Auto is a code generation engine.
This code generation engine is used by other Google projects in our database, such as Guava and Guice.
Most of the Google Auto _risky_ commits (79%) matched commits in the Guava and the Guice project.
As Guice and Guava share the same code-generation engine (Auto), it makes sense that code introducing defects in these projects share the characteristics of commits introducing defects in Auto.

### Openhab by Openhab (F$_1$-measure = 13.33%)

Openhab by Openhab provides bus for home automation or smart homes.
This is a very specific set of feature.
Moreover, Openhab and its dependencies are alone in the green cluster.
In other words, the only project against which misfire could have checked for matching defects is Openhab itself.
misfire was able to detect 2/28 bugs for Openhab.
We believe that if we had other home-automation projects in our dataset (such as _HomeAutomation_ a component based for smart home systems [@Seinturier2012]) then we would have achieved a better F$_1$-measure.

### Che by Eclipse (F$_1$-measure = 10.05%)

Eclipse Che is part of the Eclipse IDE. Eclipse provides development support for a wide range of programming languages such as C, C++, Java and others.
Despite the fact that the Che project has a decent amount of defect-commits (169) and that it is in the blue cluster (dominated by Apache) misfire was only able to detect nine _risky_ commits. After manual analysis of the 169 defect-commits, we were not able to draw any conclusion on why we were not able to achieve better performance. We can only assume that Eclipse's developers are particularly careful about how they use their dependencies and the quality of their code in general. Only 2% (169/7,818) of their commits introduce new defects.

### Annotations by Excilys (F$_1$-measure = 3.13%)

The last project we analysed manually is Annotations by Excilys.
Similar to Openhab by Openhab, it provides a very particular set of features, which consist of Java annotations for Android projects. We do not have any other project related to Java annotations or the Android ecosystem at large. This caused misfire to perform poorly.

Our interpretation of the manual analysis of the best and worst performing projects is that misfire performs best when applied to clusters that contain projects that are similar in terms of features, domain or intent. These projects tend to be interconnected through dependencies. In the future, we intend to study the correlation between the cluster betweenness measure and the performance of misfire. 

## Analysis of the Quality of the Fixes Proposed by misfire

One of the advantages of misfire over other techniques is that it also proposes fixes for the _risky_ commits it detects.
In order to evaluate the quality of the proposed fixes, we compare the proposed fixes with the actual fixes provided by the developers. To do so, we used the same preprocessing steps we applied to incoming commits: extract, pretty-print, normalize and filter the blocks modified by the proposed and actual fixes. Then, the blocks of the actual fixes and the proposed fixes can be compared with our clone comparison engine.

Similar to other studies recommending fixes, we assess the quality of the first three and five proposed fixes [@Pan2008; @Kim2013; @tao2014automatically; @Dallmeier; @le2012systematic; @le2015should]. The average similarity of the first three fixes is 44.17% while the similarity of the first five fixes is  40.78%. Results are reported in Table \ref{tab:results}.

In the framework of this study, for a fix to be ranked as qualitative it has to reach $\alpha$=35% similarity threshold.
Meaning that the proposed fixed must be at least 35\% similar to the actual fix. On average, the proposed fixes are above the $\alpha$=35% threshold.
On a per-commit basis, misfire proposed 101,462 fixes for the 13,899 true positives _risky commits_ (7.3 per commit). Out of the 101,462 proposed fixes, 78.67% are above $\alpha$=35% threshold.

In other words, misfire is able to detect _risky_ commits with 90.75% precision, 37.15% recall, and proposes fixes that contain, on average, 40-44% of the actual code needed to transform the _risky_ commit into a _non-risky_ one. 

To further assess the quality of the fixes proposed by misfire, we randomly took 250 misfire-proposed fixes and manually compared them with the actual fixes provided by the developers. For each fix, we looked at the proposed modifications (i.e., code diff) and the actual modification made by the developer of the system to fix the bug.

We were able to identify the statements from the proposed fixes that can be reused to create fixes similar to the ones that developers had proposed in 84\% of the cases. For the remaining cases, it was difficult to understand the changes that the developers made, mainly because of our lack of familiarity with the systems under study. We recognize that a better evaluation of the quality of misfire-proposed fixes would be to conduct a user study. We intend to do this as part of future work. In what follows, we present examples of misfire-proposed fixes that were detected as similar to fixes proposed by developers.

\input{tex/null.tex}

In Figures \ref{fig:null2} and \ref{fig:null1}, we show two commits that belong to the Okhttp and Druid systems, respectively. In these figures, the statements shown in red are the ones that triggered the match between the two commits. The Okhttp commit was submitted in February 2014, while the one from Druid was submitted in April 2016. The Druid commit was introduced to fix a bug, which was caused by a prior commit, submitted in March 2016. The bug consisted of invoking a function on a null reference, which led to a null pointer exception, causing the system to crash. This bug could have been avoided if the Druid developers had access to the Okhttp commit.  


In a second example, we present a case where misfire could have been used to avoid inserting a bug related to race conditions in multi-threaded code.

\input{tex/thread.tex}

In Figures \ref{fig:thread1} and \ref{fig:thread2}, we show two commits that belong to the Netty and Okhttp systems, respectively. 
For Figure \ref{fig:thread1}, we present an excerpt of the commit that triggered the match. The whole commit affected  44 files with 1,994 additions and 1,335 deletions. The Netty commit was submitted in June 2014 while the one from OKHttp was submitted in January 2017.  The bug consisted of resource leakage in a multi-threaded environment.
The similarity between the two commits comes from the \texttt{try} and \texttt{catch} blocks associated with the used exceptions, more precisely, the fact of freeing resources in case a thread crashes with a \texttt{finally} block to follow the \texttt{try} and \texttt{catch} blocks.
In the \texttt{try} block, the threads are launched and, in case an exception happens, the \texttt{catch} block is executed.
However, if the developer closes the resources consumed by the thread at the end of the \texttt{try} block then, in the case of an exception, the resources would not be freed.
Instead of duplicating the resource management code in the \texttt{try} and \texttt{catch} blocks, a good practice would be to have it in a \texttt{finally} block that always executes itself, regardless of whether an exception is thrown or not.
In the commit presented by Figure \ref{fig:thread1}, we can see that a large refactoring has been done in order to prevent crashed threads to keep using resources. This bug could have been avoided if the Okhttp developers had access to the Netty commit.  

Another example is the one depicted in Figures \ref{fig:orient} and \ref{fig:jsoup},  showing two commits that belong to the JSoup and Orientdb systems, respectively.
The first commit was submitted in November 2013, while the Orientdb was submitted two years later in October 2015. The  Orientdb commit was used to fix a bug introduced by a commit that was submitted earlier in October 2015. This bug would have been avoided if  the developer had access to the JSoup commit, that is proposed by misfire as the closest match.

In these fixes, we can see that the developers are working with the \texttt{StringBuilder} class. 
According to the Java documentation, the \texttt{StringBuilder} class \textit{provides an API compatible with StringBuffer, but with no guarantee of synchronization. This class is designed for use as a drop-in replacement for StringBuffer in places where the string buffer was being used by a single thread (as is generally the case). Where possible, it is recommended that this class be used in preference to StringBuffer as it will be faster under most implementations.}
Developers usually use the \texttt{StringBuilder} class to build strings using the \texttt{append} and \texttt{insert} methods. Using the \texttt{StringBuilder} class rather than plain string concatenation (i.e., using the \texttt{+} operator) is known to be a good Java practice as it improves performance.

In both cases, the code has been modified to avoid the appending of \texttt{null} string. In JSoup, it is done by the method \texttt{shouldCollapseAttribute}, which checks for empty values. In Orientdb, the same operation is performed by a simple null check on the string named \texttt{right}. Note that this kind of \textit{bug} would not have been spotted by a static analysis tool such as PMD [@Dangel2000] because it is \textit{legal} to pass a null string as a parameter of function expecting a string. In both cases, however, the developers were tasked to avoid the appending of null strings.

\input{tex/diff1.tex}

# Discussion {#sec:threats}

In this section we propose a discussion on limitations and threats to validity.

## Limitations

We identified three main limitations of our approach, misfire, that require further studies.

misfire is designed to work on multiple related systems. Applying misfire on a single system will most likely be 
ineffective; it is unlikely to have a large number of similar bugs within the same system. For single systems, we recommend the use of statistical models based on process and code metrics for the detection of risky commits such as the ones developed by Kamei et al. and Rosen et al. [@Kamei2013; @Rosen2015]. A metric-based solution, however, may turn to be ineffective when applied across systems because of the difficulty associated with identifying common thresholds that are applicable to a wide range of systems.    

The second limitation is related to scalability of the approach. Because misfire operates on multiple systems, we need to build a model that comprises all their commits, which is a time consuming process. It took nearly  three months using 48 Amazon Virtual Private Servers running in parallel to build and test the model for our experiments.

The third limitation we identified has to do with the fact that misfire is designed to work with Java systems only. It is however common to have a multitude of programming languages used in an environment with many inter-related systems. We intend to extend misfire to process commits from other languages as well.

## Threats to Validity

The selection of target systems is one of the common threats to validity for approaches aiming to improve the analysis of software systems. It is possible that the selected programs share common properties that we are not aware of and therefore, invalidate our results.  However, the systems analyzed by misfire were selected from Github based on their popularity and the ability to mine their past issues and to retrieve their dependencies. Any project that satisfies these criteria would be included in the analysis.  Moreover, the systems vary in terms of purpose, size, and history. 

In addition, we see a threat to validity that stems from the fact that we only used open-source systems. 
The results may not be generalizable to industrial systems. We intend to undertake these studies in future work.

The programs we used in this study are all based on the Java programming language. 
This can limit the generalization of the results to projects written in other languages. 
However, similar to Java, one can write a TXL grammar for a new language then misfire can work since misfire relies on TXL. 

Moreover, we use NICAD as the code comparison engine. The accuracy of NICAD affects the accuracy of misfire. 
This said, since NICAD has been tested on large systems, we are confident that it is a suitable engine for comparing code using TXL. Also, there is nothing that prevents from using other text-based code comparisons engines. Another threat related to the use of NICAD is the use of 35\% as a similarity threshold. A different threshold may affect the results. We chose this threshold because it resulted in best trade-off between precision and recall when  analysing a subset of the dataset. We also relied on previous studies, in which the creators of NICAD Roy et al. [@Roy2008;
@Cordy2011] showed that a threshold of around 30\% usually provides good accuracy. 

Finally, part of the analysis of the misfire proposed fixes that we did was based on manual comparison of the misfire fixes with those proposed by developers. Although we exercised great care in analyzing all the fixes, we may have misunderstood some aspects of the commits.  

In conclusion, internal and external validity have both been minimized by choosing a set of 42 different systems, using input data that can be found in any programming languages and version systems (commit and changesets).

# Conclusion {#sec:conclusion}

In this paper, we presented misfire (Bug Insertion ANticipation by Clone Analysis at commit time), an approach that detects risky commits (i.e., a commit that is likely to introduce a bug) with  an average of 90.75% precision and 37.15% recall.
misfire uses clone detection techniques and project dependency analysis to detect risky commits within and across projects.  misfire operates at commit-time, i.e., before the commits  reach the code central repository. In addition, because it relies on code comparison, misfire does not only detect risky commits but also makes recommendations to developers on how to fix them. We believe that this makes misfire a practical approach for preventing bugs and proposing corrective measures that integrates well with the developers workflow through the commit mechanism.  

To build on this work, we need to conduct a human study with developers in order to gather their feedback on the approach. 
The feedback obtained will help us fine-tune the approach. Also, we want to examine the relationship between project cluster measures (such as betweenness) and the performance of misfire. Finally, another improvement to misfire would be to support Type 4 clones.

# Reproduction Package & Dataset

As described in Section \ref{sec:newcommits}, we rely heavily on virtual machines instrumentation and coordination to run our experiments. Providing a straightforward reproduction package is therefore very challenging.
However, we are happy to share our consolidated dataset: https://github.com/MathieuNls/misfire-data.
The dataset is composed of three compressed PostgresSQL formatted tables: clones, commits and repository.
The clone table stores the relationship between set of similar commits.
The commits themselves are in the commit table with details about their author, repository, commit message and all the metrics found in commit guru [@Rosen2015].
Finally, the repository table describes the repository used in terms of url, name and ingestion status.


# Acknowledgements

We are thankful to Yves Jacquier, Olivier Pomarez, Nicolas Fleury, Alain Bedel, from Ubisoft for their participations in validating MISFIRE and particularly the fixes proposed by our approach.

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
