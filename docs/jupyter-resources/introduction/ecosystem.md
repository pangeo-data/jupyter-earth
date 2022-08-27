# Ecosystem 

_What do we mean by an "ecosystem" of software projects?_

[Wikipedia](https://en.wikipedia.org/wiki/Ecosystem) defines an ecosystem as

>  a community of living organisms in conjunction with the nonliving components of their environment, interacting as a system

A few key ideas that are common among living ecosystems and an "ecosystem" of software projects are: 
- community
- a common environment
- interaction and interoperability
- reliance and interdependence

![python-stack](../images/python-stack.png)

The term “ecosystem” is often used to describe the modern open-source scientific software. In biology, the term “ecosystem” is defined as a biological community of interacting organisms and their physical environment. Modern open-source scientific software development occurs in a similarly interconnected and interoperable fashion. The act of importing a software package signals a reliance of one part of the system on the functionality and role of another. Projects in this ecosystem lie somewhere along the spectrum from “foundational” projects which serve a wide community and are relied on by increasingly specialized projects, which eventually may be specialized enough to serve a specific scientific domain. Figure XX illustrates this concept of a software stack for the scientific python ecosystem. At the base of the stack is the language, in this case Python. Upon this sits the layer of “core” software packages. These packages provide functionality that is generically useful throughout the scientific ecosystem; for example Numpy for n-dimensional arrays {cite}`Harris2020`, Matplotlib for plotting {cite}`Hunter2007`, Pandas for data frames {cite}`reback2020pandas,mckinney-proc-scipy-2010`, and Jupyter for interactive computing {cite}`Perez2007`. The next layer consists of specialized projects such as Scikit-learn for machine learning {cite}`scikit-learn` or Dask for parallel computation {cite}`DaskDevelopmentTeam2016`; these projects are relied on by others in the ecosystem to solve a general suite of computational tasks. The top layer is that of domain-specific projects, which are tailored to serve a distinct community of researchers and the questions they are pursuing. There are not necessarily hard lines between these layers, as projects mature, they may move between these layers, and some communities might organize projects differently.

```{bibliography}
```