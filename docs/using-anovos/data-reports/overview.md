# Creating Data Reports with Anovos

The data report module is composed of two sections details of which is described further. The primary utility of keeping the two modules is to decouple the two steps in a way that one happens at a distributed way involving the **intermediate report data generation** while the other is restricted to only the pre-processing and **generation of the Anovos report**.

The reporting layer is dependent on the output produced from data analyzer modules viz. stats generator, quality checker, association evaluator alongside drift detector. 

**The report generation can happen through two ways: **
- via Config File - *Here the user specifies the desired options / input parameters as done for other modules*
- via Individual Modules - *Here the user can pick up the relevant functions specific to the reporting module and execute*
