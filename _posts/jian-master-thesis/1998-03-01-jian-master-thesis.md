---
layout: post
title:  "OSCAR - a native Object Oriented Engineering DBMS (Jian Master Thesis)"
date:   1998-03-01 00:00:00 -0500
categories: tech-database
---

## Jian's Master Thesis in Zhejiang University (浙 江 大 学 硕 士 学 位 论 文)


### Research and Implementation of the Data Definition, Data Manipulation and Schema Evolution in an OOEDBMS

#### (M.S.cs Thesis)

#### Jian Liu

#### Department of Computer Science

#### Zhejiang University

#### Hangzhou, 310027

#### P. R. China

### Abstract

This research is to develop an OOEDBMS (Object Oriented Engineering Database Management System) which is fit for practical application, especially in
the environment under STEP. Upon this, a more advanced purpose is to build an o-o data
model and to study the features that an OOEDBMS should possess.

OOEDBMS features are divided into three levels according to its importance, degree of well-
recognition and implementation. Mature parts of o-o model can be adopted in practical
implementation firstly. Meanwhile other parts of the model can get experience during real
world application to accelerate their formalization speed.

In this thesis, an object oriented data model is presented that not only suits for engineering
application using STEP but also contains more popular meaning. The data model formally
describes major features of object oriented mechanism.

The later part of the thesis introduces an OODBEMS:OSCAR, including its design and
implementation.

OSCAR is a Client/Server architecture. It not only uses EXPRESS language as modeling
language but also possesses its own DDL. These two kinds of modeling is in harmony. The
data access interface of OSCAR includes SDAI and its self-defined data manipulation
functions. C late binding is adopted to provide database facilities.

One of OODBMS essential properties is schema evolution. After referring to a lot of
literature, I take in the advantages of different schema evolution methods, design a schema
evolution method for OSCAR, and implement it. It classifies the schema modification
operations, puts forward a group of OSCAR schema invariant properties and a group of rules
which uphold the schema invariant properties. Besides, a new update/backdate mechanism
based on class versions tree and class Meta-Objects is introduced. Based on these facilities, a
requested conversion method is implemented, which propagates schema changes on instances.
This method has its advantages compared with other schema evolution methods adopted in
current OODB management systems. The results of OSCAR system runs prove that it is
really an effective, good schema evolution method.

Major features of OSCAR include:

1. Being an OOEDBMS based on product data model;
2. Supporting STEP model and information integration;
3. Adopting advanced object oriented techniques;
4. Using newest release of SDAI as data access interface;
5. Providing the facilities of version management, schema evolution, query, transaction process, view, etc. ;
6. Providing two access methods: programming and interactive interface ;
7. Having Chinese interface;
8. Owning complete copyright.


