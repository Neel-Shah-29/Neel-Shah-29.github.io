---
title: 'Community Bonding Period'
date: 12 June 2022
permalink: /posts/2012/08/Community-Bonding-Period/
---

# Community Bonding Period

<div align="center">
<img src="https://user-images.githubusercontent.com/84740927/170821829-f631f5f7-c410-429a-acdc-d9cc81d13dd3.jpeg" alt="drawing" width="500"/>
</div>


This is the period of time between when accepted GSoC contributors are announced and the time they are expected to start coding. This time is an excellent one to introduce your GSoC contributors to the community, get them on the right mailing lists, introduce them to the codebase, discuss how they will work with their mentors on their timeline for the program, etc.

I had an unique experience during the community bonding period, i will share my experience and the kind of satisfaction i got after the completion of this period.

## Why is Community Bonding Period so important?

<div align="center">
<img src="https://miro.medium.com/max/1200/1*YvvzCmO4dbCPeTPOq7_OKA.png" alt="drawing" width="500"/>
</div>


In this period we make connections with the people who have been involved with the organization for long (mentor and student relationship). We indulge in several meetings scheduled almost daily to understand the requirements of the project and how do they align with our interests. This period lasts for 3 weeks which is a pretty good amount of time. I had a great conversation with my mentors even before Google announced the list of selected students. We had already discussed the plans on how we are going to do things.

But before the community bonding period started, I looked at the huge codebase of TMVA, and i was feeling like its impossible for me to do this project, as i was not firm enough with the codebase.
<div align="center">
<img src="https://user-images.githubusercontent.com/84740927/176964361-7fd46687-5eac-488a-aaa4-57dbb005ef46.png" height="250" width="350">
</div>

All the organisations have their own rules , own planning and an independent list of instructions for submissions and coding throughout the journey. During this period i got familiar with my mentor more,we used to talk almost everyday, I planned my project and timetable, with him. We receive guidelines for contribution, and i asked for resources to gain knowledge on project topics, etc.

Basically CERN had a mailing list for all the updates of GSOC 2022 and Mattermost channel for communication with the mentors.

So, basically this is the most crucial period to set up the environment before you get into the actual coding period. I would say not to waste this time and utilize this time efficiently to get a headstart in your project.

### Timeline of Community Bonding Period for GSOC 2022

This year, it started on 20th May. During this time students receive official letters from Google and the community. Students are asked to set up their Payoneer account to receive the stipend. They receive various links to get connected for communication. June 12 was the day when it ended and the official coding period started.

## Work done in Community Bonding Period

<div align="center">
<img src="https://miro.medium.com/max/700/1*lZEfeZ5_IeYJ1FqA8Izk1Q.jpeg" alt="drawing" width="500"/>
</div>


- Got familier with the huge codebase of Root/TMVA which was highly recommended for my project. [Here](https://github.com/root-project/root) is the link to repository.  

- Ask for the resources to mentor required for the project. Some of them are mentioned in my previous blog. Here is the list of other additional resources.

  **--->** Check out this [repository](https://github.com/axmat/TMVAInference). Here the operators are implemented by running the inference directly. By doing so it was easier to test the implementation before translating it into the code generation format.

  **--->** Understanding how multidimensional arrays (vector, matrix, 3D and 4D tensor) are represented in memory : 

  [https://oneapi-src.github.io/oneDNN/understanding_memory_formats.html](https://oneapi-src.github.io/oneDNN/understanding_memory_formats.html)

  **--->** BLAS and basic numerical linear algebra (matrix multiplication, dot product, ...) : 

  1. [https://stackoverflow.com/questions/1303182/how-does-blas-get-such-extreme-performance](https://stackoverflow.com/questions/1303182/how-does-blas-get-such-extreme-performance)

  2. [http://www.netlib.org/utk/people/JackDongarra/WEB-PAGES/SPRING-2005/Lect07.pdf](http://www.netlib.org/utk/people/JackDongarra/WEB-PAGES/SPRING-2005/Lect07.pdf)

  **--->** Advanced, if you have time try to learn or review cache misses, blocking (for matrix multiplication), SIMD, time complexity of algorithms

- For TMVA projects we usually have weekly meetings to discuss about the projects and solve the difficulties of everyone. So attending those meetings are mandatory as it involves your communication with other gsoc students of TMVA. Also the round table discussion solves all queries which you have as all the students and mentors give solutions and guidelines on it. Additionally we can also set a meeting with the mentor if required.

- In TMVA projects, we have a specific idea of presenting our projects with the detailed timeline of implementation in front of mentors and other Gsoc students so we can get familier with other's project as well and by that time our concrete strategy of project is also ready.

  Here is my presentation for reference:- [Inference Code Generation for Deep Learning models](https://docs.google.com/presentation/d/1bl1Xk5esr5TJEHKoro7o05pRQ2J997Qb1hc68XfU_0s/edit?usp=sharing)

- I built and set up the Root environment in my laptop as required. [Here](https://root.cern/install/build_from_source/) is the link to build Root from source.

- Finalised the evaluation dates so that i get enough time to complete my long term project. I chose a 16 weeks project after discussing with the mentor as my college reopens from July 18 and i will get less time to devote after that. 

- Lastly, I began coding part for one of the easy operator which is Leaky Relu so i get an headstart for the Project. Here is the link to the PR :- [Adding a Leaky Relu ONNX Operator](https://github.com/root-project/root/pull/10415)

These are important things to be done, also there can be various other non-coding GSoC related things which you can clear out during this period.

Hope you all have got a clear idea about the community Bonding Period and its advantages in GSOC.Thanks for reading!
I will be back with another interesting blog soon. 
Until then, Goodbye!

**Thanks and Regards,**

**Neel Shah** 
