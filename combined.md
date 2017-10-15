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

\IEEEPARstart{R}{esearch} in software maintenance has evolved over the year to include areas like mining bug repositories, bug analytic, and bug prevention and reproduction. The ultimate goal is to develop better techniques and tools to./ help software developers detect, correct, and prevent bugs effectively and efficiently. 

One particular (and growing) line of research focuses on the problem of preventing the introduction of bugs by detecting risky commits (preferably before the commits reach the central repository). Recent approaches (e.g., [@Lo2013; @Nam2013]) rely on training models based on code and process metrics (e.g., code complexity, the experience of the developers, etc.) that are used to classify new commits as risky or not. 

Despite the recent advances in the field, the literature shows that many existing software maintenance tools have yet to be adopted by industry [@Lewis2013; @Foss2015; @Layman2007; @Ayewah2007; @Ayewah2008; @Johnson2013; @Norman2013; @Hovemeyer2004; @Lopez2011]. After assessing existing commercially available tools and state of the art approaches proposed in the litterature with software architects, team leaders, and senior developers, we extracted factors that we believe are contributing to this situation:

- Integration with the developer's workflow: Most existing maintenance tools ([@Kim2006a; @Ayewah2008b; @Findbugs2015; @Moha2010; @Palma; @Nayrolles2013d; @Nayrolles; @Nayrolles2013a;  @Nayrolles2015a] are some noticeable examples) are not integrated well into the work flow of software developers (i.e., coding, testing, debugging, committing). Using these tools, developers have to download, install and understand them to achieve a given task. They would constantly need to switch from one workspace to another for different tasks (i.e., feature location with a command line tool, development and testing code with an IDE, development and testing front end code with another IDE and a browser, etc.) [@Robertson2004; @Robertson2006; @Beckwith2006].

- Corrective actions: The outcome of these tools does not always lead to corrective actions that the developers can implement. Most of these tools return several results that are often difficult to interpret by developers. Take, for example, FindBugs [@Hovemeyer2004], a popular bug detection tool. This tool detects hundreds of bug signatures and reports them using an abbreviated code such as `CO_COMPARETO_INCORRECT_FLOATING`. Using this code, developers can browse the FindBug's dictionary and find the corresponding definition _This method compares double or float values using a pattern like this: `val1 > val2 ? 1 : val1 < val2 ? -1 : 0`_. While the detection of this bug pattern is accurate, the tool does not propose any corrective actions to the developers that can help them fix the problem. Moreover, it has been reported in the literature that the output of existing maintenance tools tends to be verbose at the point where developers decide to simply ignore them [@Arai2014; @Kim2007b; @Kim2007c; @Ayewah2010; @Shen2011].

- Leverage of historical data: These tools do not leverage a large body of knowledge that already exists in open source or within the organisation.  For defect prevention, for example, the state of the art approaches consists of adapting statistical models built for one project to another project [@Lo2013; @Nam2013].	As argued by Lewis _et al._ [@Lewis2013] and Johnson _et al._ [@Johnson2013], approaches based solely on statistical models are perceived by developers as black box solutions. Developers are less likely to trust the output of these tools.

In this paper, we propose an approach that adresses these challenges whithin the framework of ultra-large repositories at Ubisoft.

Ubisoft, one of the world's largest video game development companies, specialises in the design and implementation of high-budget video games. 
Such high-budget video games involve thousands of highly qualified personels from various fields (developers, artists, management, marketing, ...).
To give the reader a grasp over the actual size of such software projects; the final products can contain millions of files and hundreds of thousands of commits.   
Furthermore, several high-budget games are under development at the same time by 8,000 developers scatered accross the 29 locations on six continents. 

Our approach named MISFIRE (MetrIcS and clone detection for Fault Interception and Removing in ultra-large rEpositories) leverages three decades of code history in a cross-projects manner with the aims to automatically detecte and propose faults correction before they reach the central software repository. 
More specifically, it uses a combination of well-known code metrics to build statistical models to prevent faults insertion. In addition to these metrics, we also use an in-house clone detector that extracts code blocks from incoming commits and compare them to those of known defect-introducing commits. Finally, using the fix used to fix past defects, we can provide a solution, in the form of a contextualised diff, to the developer that would remove the fault. A contextualised diff is a diff that is tempered with to match the current workspace of the developer regarding variable types and names.

This novel approach outperforms state-of-the-art approaches such as Commit Guru [@Rosen2015] when it comes to defect introduction detection. Indeed, we can detect defects introduction with a 79% precision and a 65% recall. The performances of Commit-Guru, on the same dataset, are 66% precision and 63% recall.
Also we asked in-house experts to manually evaluate the quality of the fix we propose to remove the faults from the current code submission. 
More than XX% of the analysed proposed fixes were judged highly relevant, and another XX% were classified as relevant. 
Overall, XX% of the proposed fixes help developers toward the craft of an actual fix.
Finally, MISFIRE addresses the three factors we identified as contributing to the slow adoption of automatic defect prevention tools in large companies (Integration with the developer's workflow, Corrective actions and Leverage of historical data).

The remaining parts of this paper are organised as follows. In Section \ref{sec:relwork}, we present related work. Sections \ref{sec:misfire},  \ref{sec:exp} and \ref{sec:result} are dedicated to the misfire approach, the case study setup, and the case study results.
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

Our work shares a similar goal to works on the prediction of risky changes. However, misfire takes a different approach in that it leverages dependencies of a project to determine risky changes.


## Transfert Defect Learning

\mathieu{To do after we complete V.C}

## Automatic Patch Generation

Since misfire not only flags risky changes but also provides developers with fixes that have been applied in the past, automatic patch generation work is also related. Pan *et al.* [@Pan2008] identified 27 bug fixing patterns that can be applied to fix software bugs in Java programs. They showed that between 45.7 - 63.6\% of the bugs can be fixed with their patterns. Later, Kim *et al.* [@Kim2013] generated patches from human-written patches and showed that their tool, PAR, successfully generated patches for 27 of 119 bugs. Tao *et al.* [@tao2014automatically] also showed that automatically generated patches can assist developers in debugging tasks. Other work also focused on determining how to best generate acceptable and high quality patches, e.g. [@Dallmeier; @le2012systematic], and determine what bugs are best fit for automatic patch generation [@le2015should].

Our work differs from the work on automated patch generation in that we do not generate patches, rather we use clone detection to determine the similarity of a change to a previous risky change and suggest to the developer the fixes of the prior risky changes.

# The MISFIRE Approach {#sec:misfire}

Figures \ref{fig:misfire1}, \ref{fig:misfire3} and \ref{fig:misfire2} show an overview of the misfire approach, which consists of two parallel processes. 

In the first process (Figures \ref{fig:misfire1} and \ref{fig:misfire3}), misfire manages events happening on project tracking systems to extract defect-introducing commits and commits that provided the fixes. For simplicity, in the rest of this paper, we refer to commits that are used to fix defects as _fix-commits_. 
We use the term _defect-commit_ to mean a commit that introduces a defect. In the second phase, misfire analyses the developer's new commits before they reach the central repository to detect potential risky commits (commits that may introduce bugs). 
Also, MISFIRE proposes potential fixes mined from the all the repositories within the organisation.
Moreover, the fixes are transformed to match the actual workspace of the developer using adpating variable names, indentation and overall code structure.

\input{tex/approach}

The project tracking component of misfire listens to bug (or issue) closing events of major open-source projects (currently, misfire is tested with 12 large projects within Ubisoft). These projects share many dependencies that are both internal and external. We perform project dependency analysis to identify groups of highly-coupled projects with the aim to be able to transfert defect learning from one project to another similar project within the organisation. 
Transfering defects learning from projects to projects is a challenging task [@Nam2013], and, as shown in our experiments (Section \ref{sec:exp}) this adhoc metric gives us statisfactory results. 

In the second process (Figure \ref{fig:misfire2}), MISFIRE intercepts incoming commit before they leave developers' workstation using a pre-commit hook. 
A pre-commit hook is a script that executes itself at commit-time.
Pre-commit hooks are custom scripts set to fire off when certain important actions of the versioning process occur.
There are two groups of hooks: client-side and server-side. Client-side hooks are triggered by operations such as committing and merging, whereas server-side hooks run on network operations such as receiving pushed commits. These hooks can be used for all sorts of reasons such as checking compliance with coding rules or automatic runs of unit test suites. The pre-commit hook runs before the developer specifies a commit message.
At Ubisoft, several actions are undertook at commit-time.
For example, one has to refer which task or issue is being addressed by the commit at hand, specify which reviewers reviewed the commit at hand if it was done during a pair-programming session and so on.
We seamlessly intergrate MISFIRE with this already existing process.
First, we extract the block of code modifed by the developer and compute the metrics representing the commit at hand. 
The metrics are exhaustively described in section \ref{sec:offline}.
Using the resulting vectore (step 4), we classify the commit as _risky_ or _non-risky_. 
A _risky_ commit is a commit that is likely to introduce a defect while a _non-risky_ commit is classified as sane.
Note that false positives (sane commit classified as _risky_) and false negatives (commit introducing a defect classified as _non-risky_) are present in this step.
In section \ref{sec:exp}, we report these results and explain how we favor precision (i.e. reduce the amount of false-positives) over recall (i.e. reduce the amount of false-negative) to create a sense of trust between developers and MISFIRE. 
We report on this particular aspect in section \ref{sec:threats}.
If in step 4 (classification) the commit is classified as _non-risky_, then the process stops, and the commit is allowed to be transfered from the developer's workstation to the central repository.
In the case of a _risky_ classification, the process continues with further analyses. 
We first start by normalising and formatting blocks of modified and compares them to the known defect-introducing commits present in the currated cluster by using text-based type 2 and type 3 clone detection.
The clusters are said to be currated because we collect usage statics when MISFIRE proposes a contextualised fix to the developer. If, the developer does not use the fix the pertinence score ($p-score$) of the fix is reduced and we only propose fixes that have a $p-score >\alpha$. $\alpha$ is a live metric that management teams can adjust at any time without the need to rebuild statistical models.
In addition, $\alpha$ is automatically adjusted in a nightly fashion based on the usage statistics of the day before.
Finally, $\alpha$ can be different across projects, teams and even individual developers.
In step 7, we adapt the matchin fixes to the actual context of the developer by modifying indentation depth and variable name in an effort to reduce context switching [@Code; @Danny1997; @Altmann2004].
Finally, depending on the similarity several processes can be engaged.
For example, if the similarity between the commit at hand and known defect-introducing commit in the same project's cluster is $S>80\%$ then, we perform a hard reject of the commit while suggesting how to improve it. 
If the similarity is over 50%, then we perform a soft-reject where the developers would have to seek an aditional review before being able to submit the commit to the central repository.
Finally, if the similarity is over 30%, then we accept the commit and suggest potential improvements throught our contextualised fixes.
As for $\alpha$, these thresholds for $S$ can be different by projects, teams and individuals and are manually and automatically adjustable.

In the upcoming subsections, we describe each step in detail.

## Clustering Project Repositories {#sec:clustering}

We cluster projects according to their dependencies.  The rationale is that projects that share dependencies are most likely to contain defects caused by misuse of these dependencies. In this step, the project dependencies are analysed and saved into a single NoSQL graph database as shown in Figure \ref{fig:misfire3}. Graph databases use graph structures as a way to store and query information.  In our case,  a node corresponds to a project that is connected to other projects on which it depends. 
Dependencies can be _external_ or _internal_. _external_ dependencies refer to products that are not creater nor maintained within the organisation.
These products can be open and closed source both.
_Internal_ dependencies refer to projects that are maintained in-house.
While we cannot describe in details the _external_ and _internal_ dependencies used at Ubisoft for confidentiality and security reasons, we can offer the reader a graphical representation of the 12 analysed projects (yellow) with their dependencies (blue) \ref{fig:dep-graph} and the resulting clusters \ref{fig:network-sample}. 

\input{tex/dependencies}

Internal dependencies are managed within the framework of a single repository which makes their automatic extraction possible.
The dependencies could also be automatically retrieved if projects use a dependency manager such as Maven. 

\input{tex/network-sample}

Once the project dependency graph is extracted, we use a clustering algorithm to partition the graph. To this end, we choose the Girvan–Newman algorithm [@Girvan2002; @Newman2004], used to detect communities by progressively removing edges from the original network. Instead of trying to construct a measure that identifies the edges that are the most central to communities, the Girvan–Newman algorithm focuses on edges that are most likely "between" communities. This algorithm is very effective at discovering community structure in both computer-generated and real-world network data [@Newman2004]. Other clustering algorithms can also be used. 

The clusters are then used to devide the known defect introducing commits and their associated fixes. 
When in search for a solution to a _risky_ commit we will only look for solutions applied to defect introducing commit in the same cluster.
This allows to reduce the search space while enhancing the quality of the proposed solution as belonging to the same cluster is a mark of intrinsec similarity between projects regarding dependencies. 



## Building a Database of Code Blocks of Defect-Commits and Fix-Commits {#sec:offline}

To build our database of code blocks that are related to defect-commits and fix-commits, we first need to identify the respective commits. Then, we extract the relevant blocks of code from the commits.

**Extracting Commits:** Misfire listens to bug (or issue) closing events happening on the project tracking system. Every time an issue is closed, misfire retrieves the commit that was used to fix the issue (the fix-commit) as well as the one that introduced the defect (the defect-commit). Retrieving fix-commits, however, is known to be a challenging task [@Wu2011]. This is because the link between the project tracking system and the code version control system is not always explicit. In an ideal situation, developers would add a reference to the issue they work on inside the description of the commit. However, this good practice is not always followed. To link fix-commits and their related issues we implemented the SZZ algorithm [@Kim2006c]. In addition to the SZZ algorithm, we build a statistical model using the following code change metrics:

- la: lines added
- ld: lines deleted
- nf: Number of modified files
- ns: Number of modified subsystems
- nd: number of modified directories
- en: distriubtion of modified code across each file
- lt: lines of code in each file (sum) before the commit
- ndev: the number of developers that modifed the files in a commit
- age: the average time interval between the last and current change
- exp: number of changes previously made by the author 
- rexp: experience weighted by age of files (1 / (n + 1))
- sexp: previous  changes made by the author in the same subsystem
- loc: Total modified LOC across all files
- nuc: number of unique changes to the files

Finally, a web api is able to receive new commits and provide a way to access to the statistical model 

It already exists an open-source implementation of such a system developed by Rosen _et al._ called Commit-Guru [@Rosen2015].
More specifically, we ported a Python versio developed by Rosen _et al._  [@Rosen2015] in GoLang for performances and internal maintenability reasons.

As Commit-guru's back-end, we have has three major components: ingestion, analysis, and prediction. The ingestion component is responsible for ingesting (i.e., downloading) a given repository.
Once the repository is entirely downloaded on a local server, each commit history is analysed. 
Unlike commit-guru that classifies commit using the list of keywords proposed by Hindle *et al.* [@Hindle2008], MISFIRE classifies the commit using the internal project tracking system.
Project tracking system allows one to report unexpected system behaviour and managers can assign them to developers.
Using the internal pre-commit hook, developers must link every commit to a give task #ID. 
If the task #ID entered by the developer matches a bug or crash report within the project tracking system, then we perform the SCM blame/annotate function on all the modified lines of code for their corresponding files on the fix-commit's parents. This returns the commits that previously modified these lines of code and are flagged as the defect introducing commits (i.e., the defect-commits).

The SZZ algorithm, used by commit-guru and MISFIRE has been shown to be effective in detecting risky commits [@Kamei2013; @Rosen2015]. 
Moreover, we are more accurate in identfying fixing commits than any approaches using a keyword classification [@Hindle2008] and we do not rely on linking tools to re-construct the relationship between the commit and an issue such as Relink [@Wu2011] as 100\% of the commits are linked to tasks by design.

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
We do so by checking the block's beginning and ending with a parentheses algorithms [@bultena1998eades]. 

One important note about this database is that the process can be cold-started.
A tool supporting misfire does not need to *wait* for a project to have issues and fixes to be in effect.
It can leverage the defect-commits and fix-commits
of projects in the same cluster that already have a history.
Therefore, misfire is applicable at the beginning of every
project. The only requirement is to use a dependency manager. 

## Finding potential fixes for faults  {#sec:online}

Each time a developer makes a commit, misfire intercepts it using a pre-commit hook,  extracts the corresponding code block (in a similar way as in the previous phase), and compares it to the code blocks of historical defect-commits. If there is a match, then the new commit is deemed to be risky. A threshold $S$ is used to assess the extent beyond which two commits are considered similar. 

To compare the extracted blocks to the ones in the database, we resort to clone detection techniques, more specifically, text-based clone detection techniques. This is because lexical and syntactic analysis approaches (alternatives to text-based comparisons) would require a complete program to work, i.e., a program that compiles.  In the relatively wide-range of tools and techniques that exist to detect clones by considering code as text [@Johnson1993;  @Johnson1994; @Marcus; @Manber1994; @StephaneDucasse; @Wettel2005; @Cordy2011] we had to build our own text-based clone detector for several reasons.
First and foremost, the clone detector should have the ability; once a clone has been indentified, to trasnform the matching clones for them to match the workspace of the developer regarding variables names and data structure.
While classical clone detector aims to detect clone pairs for removal and/or managing them, we have another goal.
Indeed, we want non only to match clone pairs by abstracting them; we also want to transform one blocks into another.
The contextualised fix that we proposed to the developers can be syntactically incorect as they are based on imcomplete blocks of code.
Hence, they cannot be used directly by developers. 
We simply believe that the contextualised fixes are easier to understand and apply as the _look_ familiar to the code the developer is currently attempting to submit.

Our clone detector can detect Types 1, 2 and 3 software clones [@CoryKapser]. Type 1 clones are copy-pasted blocks of code that only differ from each other in terms of non-code artefacts such as indentation, whitespaces, comments and so on.  Type 2 clones are blocks of code that are syntactically identical except literals, identifiers, and types that can be modified. Also, Type 2 clones share the particularities of Type 1 about indentation, whitespaces, and comments. Type 3 clones are similar to Type 2 clones in terms of modification of literals, identifiers, types, indentation, whitespaces, and comments but also contain added or deleted code statements. misfire detects Type 3 clones since they can contain added or deleted code statements, which make them suitable for comparing commit code blocks.

For our clone detector, we reuse the pretty-printing strategy from Roy _et al._ where  statements are broken down into several lines [@Iss2009].
Furthermore, in th process, statements can be  shows how this can improve the accuracy of clone detection with three `for` statements `for (i=0; i<10; i++)`,  `for (i=1; i<10; i++)` and `for (j=2; j<100; j++)`.

The pretty-printing allows us to detect Segments 1 and 2 as a clone pair, as shown by Table \ref{tab:pretty-printing}, because only the initialisation of $i$ changed. This specific example would not have been marked as a clone by other tools we tested such as Duploc [@StephaneDucasse]. 

\input{tex/Pretty-Printing}

The extracted, pretty-printed, normal ised and filtered blocks are marked as potential clones using a Longest Common Subsequence (LCS) algorithm [@Hunt1977]. Then, a percentage of unique statements can be computed and, given the threshold $\alpha$, the blocks are marked as clones.

Another important aspect of the design of misfire is the ability to provide guidance to developers on how to improve risky commits. We achieve this by extracting from the database the fix-commit corresponding to the matching defect-commit and present it to the developer. We believe that this makes misfire a practical approach for the developers as they will know why a given modification has been reported as risky in terms of code; this is something that is not supported by techniques based on statistical models (e.g., [@Kamei2013a; @Rosen2015]).  
A tool that supports misfire should have enough flexibility to allow developers to enable or disable the recommendations made by misfire. 
Furthermore, because misfire acts before the commit reach the central repository, it prevents unfortunate pulls of defects by other members of the organisation. 

While we cannot provide actual defect introducing and bug fixing commits, we provide a fictional example using two version of the well known bubble sort algorithm in table \ref{tab:bubble}.
In the fictional example, we assume that the bubble sort on the left column would be a fix of the bubble sort on the right column.
First, both bubble sorts are pretty-printed.
Then, the variables and types are abstracted using _#_ signs.
Note that we display complete abstraction, but we can also choose to abstract only the variables names and keep the variables types.
Both abstractions are performed by our clone detector and each commit classified as _risky_ are compared to both abstractions.
Then, the parts that match in both abstraction are displayed in red.
Here, we have 10/19 lines that match.
Consequently, the similarity between the two code abstracted code blocks is 52.6\%.
Finally, in the third row, we display the original fix and the contextualised one.
On the contextualised fix, the variables names and types have been modified so they match the code that the developer is currently trying to submit.

## Classifying Incoming Commits

The classification of incoming commits within Misfire is two folds.
First, the commit goes throught the statical part of the approach (steps 1 to 4). 
If the statical part of the approach classifies the commit as _non-risky_, we simply let it through, and we stop the processing here.
If the commit is classified as _risky_, however, we continue the process with the steps 5 to 9.
It is important to note that we do not output a _risky_ classification if we are unable to find a match in known defect-introducing signatures in our history.
This serves three different goals.
The first one is to improve the precision, in terms of false positives, of the approach.
It is our opignion that, as an organisation, such tools must maintain an acceptable level of trust from its users.
If a _risky_ commit classifier emits too many false alarms (i.e. classifying sane commit as _risky_); then it risks loosing the trust of its users and being ignored altogether.
The second goals behind this double validation is to be able to propose a solution to the _risky_ commit for each _risky_ classification.
Indeed, one of the shortfall we identified of existing tool is their inability to provide adequate actions to fix the problem at hand.
In our case, each alarm is accompagnied with a contextualised changeset that serves as a potential solution.
Finally, this two-steps classification allow us to significantly reduce the time required, in average, to classify a commit.
Indeed, if each commit were to be analyzed against all the known signature in terms of code clone similarity, then, it would take more than 268 seconds per commit using the cluster we describe in the next section while analysing a normal day worth of commit.
While 268 seconds, in average, could be perceived as acceptable for a classification, it is our opinion that such system should be near-instanteneous to be used.
With the two steps validation, it takes, on average, 3.75 seconds.

In short, the approach is designed to have the highest precision possible while maitaining an acceptable recall and provide hints towards the solution in a near-instanteneous manner.


# Case Study Setup {#sec:exp}

In this section, we present the setup of our case study in terms of repository selection, dependency analysis, comparison process and evaluation measures.

## Project Repository Selection {#sec:rep}

To select the projects used to evaluate our approach, we followed one simple criterion. Each project must be a major project within the organisation (i.e. AAA games) and not a library. 
In addition, each project is based on the same game engine. 
Ubisoft does have many game engine for different kind of needs.

## Project Dependency Analysis {#sec:dependencies}

Figure \ref{fig:dep-graph} shows the project dependency graph. 
As shown in Figure \ref{fig:dep-graph}, these projects are very much interconnected. 
A review of each cluster shows that this partitioning divides projects in terms of high-level functionalities. For example, one cluster is related to a particular given family of video games while the other can be another family.
11 engineers have manually validated these clusters at Ubisoft, and they are accurate at grouping similar projects together.

## Building a Database of Defect-Commits and Fix-Commits for Performances Evaluation {#sub:golden}

To build the database that we can use to assess the performance of Misfire, we use the same process as discussed in Section \ref{sec:offline}.
We retrieve the full history of each project and label commits as defect-commits if they appear to be linked to a closed issue using the SZZ algorithm [@Kim2006c].
Then, this baseline is used to compute the precision and recall of Misfire. Each time Misfire classifies a commit as _risky_; we can check if the _risky_ commit is in the database of defect-introducing commits. 

The same evaluation process is used by related studies  [@ElEmam2001; @Lee2011a; @Bhattacharya2011; @Kpodjedo2010;@Kamei2013].


## Process of Comparing New Commits {#sec:newcommits}

Because our approach relies on commit pre-hooks to detect risky commits, we had to find a way to *replay* past commits. 
To do so, we *cloned* our test subjects, and then created a new branch called *misfire*. When created, this branch is reinitialized at the initial state of the project (the first commit), and each commit can be replayed as they have originally been.  For each commit, we store the time taken for *misfire* to run, the number of detected clone pairs, and the commits that match the current commit. 
As an example, let's assume that we have three commits from two projects. 
At time $t_1$,  commit $c_1$ in project $p_1$ introduces a defect. 
The defect is experienced by a user that reports it via an issue $i_1$ at $t_2$.
A developer fixes the defect introduced by $c_1$ in commit $c_2$ and closes $i_1$ at $t_3$.
From $t_3$ we known that $c_1$ introduced a defect using the process described in Section \ref{sub:golden}.
If at $t_4$, $c_3$ is pushed to $p_2$ and $c_3$ matches $c_1$ after preprocessing, pretty-printing and formatting, then $c_3$ is classified as _risky_ by misfire and $c_2$ is proposed to the developer as a potential solution for the defect introduced in $c_3$.

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

In this section, we show the effectiveness of misfire in detecting risky commits using clone detection and project dependency analysis. The main research question addressed by this case study is: _Can we detect risky commits by combining metrucs and code comparison within and across related projects, and if so, what would be the accuracy?_

The experiments took nearly two months using a cluster of six 12 3.6 Ghz cores with 32GB of RAM. The longest part of the experiment is to construct the baseline as each commit must be analysed with the SZZ algorithm. Once the basline was established, the model built, it took, on average, 3.75 seconds to analyse an incoming commit on our cluster.

In the following subsections, we provide insights on the performance of Misfire by comparing it to several other classifiers. 
We compare it to a Commit-guru [@Rosen2015] as it is a good indicator of what a statical model only approach can achieve in terms of performances and it is accuracy have been proven.
Then, we compare Misfire with itself but without leveraging our clusters by dependencies. 
In other words, we applied the same approach again but without integrating the defects from other projects in the statistical model nor the clone detection.
This allows us to validate the usefulness of our clusters and, consequently, validate the approach for large multi-projects ecosystems.

## Performance of Misfire 

This novel approach outperforms state-of-the-art approaches such as Commit Guru [@Rosen2015] when it comes to defect introduction detection. Indeed, we are able to detect defects introduction with a 79.10% precision and a 65.61% recall in average on the 12 projects we analysed at Ubisoft so far while using the threshold displayed on \ref{fig:misfire2}.
Any commit satisfying the two-steps classification but have a less than 30\% will be classified as _non-risky_. 
On the same dataset Commit-Guru 66.71% precision and 63.01% recall.
While applying only the first step of classification (i.e. the stastical one) and not code clone comparison, then, Misire obtains 70.05% precision and 71.40% recall.

It is important to note that we did not evaluate the effect of the feedback loop, where developers indicate if they found the proposed fix applicable, on the precision and the recall.
While Misfire is currently beta-tested by the teams of one product, we need to gather at least a year of utilisation to measure the impact.
We intent to report of this as a future work.
We could expect the precision to go up and the recall to go down as bugs signatures are discarded from the cluster if no new bugs signatures were to be added.
Indeed, it is unclear what the effect of adding signature will discarding other over time will be. 

## Cluster Classifier Performances

Our clusters, computed with the dependencies of each project allow to solve one major problem in defect learning and prediction: cold start [@Schein2002].
Several approaches have already been proposed to conteract these problems, however, they all require to classify _similar_ project by hand and then, maninipulate the feature space in order to adapt the model learnt from one project to the other [@Nam2013]. 
With our approach, the _similarity_ between projects is computed automatically and, results show that we do not actually need to maninipulate the feature space.
Figures \ref{fig:bluecluster}, \ref{fig:yellowcluster} and \ref{fig:redcluster} show the performances of the Misfire classification, in terms of ROC-curve and recall over precision curves, for the first thousand commit of the last project (chronologically) for the blue, yellow and red clusters presented in Figure \ref{fig:network-sample}.
The left sides or the graph are low cutoff (aggressive) while the right sides are high cutoff (conservative).
The area under the ROC curves are 0.817, 0.763 and 0.806 for the blue, yellow and red clusters, respectively.


\input{tex/clusterRoc}

These results show that not only the clusters are efficient in identfying similar projects for defect learning transfert but also provide excellent performances while starting a new project.
As new projects mature, their defects would be integrated into the model in order to improve it.

To confirm this, we ran an experiment with the blue cluster where we first apply the model learnt from other members of the cluster for the first thousand commits.
The performance of this model is 75.1% precision, 57.6% recall for the first thousand commits.
After the first thousand commits, we take the commits of the day (for each day until the end of the project) and rebuild the model by adding the commits of the day to the ones already known from the clusters.
Overall, combining the commits from the project at-hand in a nightly fashion with the commits known from the cluster allowed us to correctly classified an additional 2.05\% of commits in the last 30\% of the project life compared to only using the commits from the project. 
This shows that, in addition to providing a viable alternative to a cold-start, our clusters also allow us to enhance the performances of the model at all time and no only at the start.

## Analysis of the Quality of the Fixes Proposed by Misfire

\mathieu{This is plannified this week with architects at Ubisoft}

# Discussion {#sec:threats}

In this section, we propose a discussion on limitations and threats to validity.

## Limitations

We identified two main limitations of our approach, misfire, which require further studies.

Misfire is designed to work on multiple related systems. Applying misfire on a single system will most likely be less effective as the the two steps classification would be hindered by the fact that it is unlikely to have a large number of similar bugs within the same system. For single systems, we recommend the use of statistical models based on process and code metrics for the detection of risky commits such as the ones developed by Kamei et al. and Rosen et al. [@Kamei2013; @Rosen2015]. A metric-based solution, however, may turn to be ineffective when applied across systems because of the difficulty associated with identifying common thresholds that are applicable to a wide range of systems.    

The second limitation we identified has to do with the fact that misfire is designed to work with Ubisoft systems. Ubisoft uses C\#, C, C++, Java and other internally developped languages. It is however common to have  other languages used in an environment with many inter-related systems. We intend to extend misfire to process commits from other languages as well.

## Threats to Validity

The selection of target systems is one of the common threats to validity for approaches aiming to improve the analysis of software systems. It is possible that the selected programs share common properties that we are not aware of and therefore, invalidate our results. Howerver, the systems vary in terms of purpose, size, and history. 

In addition, we see a threat to validity that stems from the fact that we only used closed-source systems. 
The results may not be generalizable to open-source systems.

The programs we used in this study are all based on the C\#, C, C++ and Java programming language. 
This can limit the generalisation of the results to projects written in other languages. 

Finally, part of the analysis of the misfire proposed fixes that we did was based on manual comparisons of the misfire fixes with those proposed by developers with a focus group composed of experienced engineers and software achitects. Although we exercised great care in analysing all the fixes, we may have misunderstood some aspects of the commits.  

In conclusion, internal and external validity have both been minimized by choosing a set of 12 different systems, using input data that can be found in any programming languages and version systems (commit and changesets).

# Conclusion {#sec:conclusion}

In this paper, we presented misfire (MetrIcS and clone detection for Fault Interception and Removing in ultra-large rEpositories), an approach that detects risky commits (i.e., a commit that is likely to introduce a bug) with an average of 79.10% precision and a 65.61% recall.
Misfire combines code metrics, clone detection techniques and project dependency analysis to detect risky commits within and across projects.  Misfire operates at commit-time, i.e., before the commits reach the central code repository. Also, because it relies on code comparison, misfire does not only detect risky commits but also makes recommendations to developers on how to fix them. We believe that this makes misfire a practical approach for preventing bugs and proposing corrective measures that integrate well with the developer's workflow through the commit mechanism.  

# Reproduction Package

For security and confidentiality reasons we cannot provide a reproduction package that will inevatibly involve Ubisoft's copywrited source code.
However, the Misfire source code is in the process of being open-sourced and will be soon available at https://github.com/ubisoftinc.

# Acknowledgements

We are thankful to Yves Jacquier, Olivier Pomarez, Nicolas Fleury, Alain Bedel, Mark Besner, David Punset, Paul Vlasie, Cyrille Gauclin, Luc Bouchard and Chadi Lebbos from Ubisoft for their participations in validating MISFIRE hypothesis, efficiency and the fixes proposed by our approach.

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

