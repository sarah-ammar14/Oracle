# TP2

Vous trouverez dans ce [lien](https://docs.google.com/presentation/d/1f5uyqowZ7u9QAV5YUseURFAPnu4v6_O7cnL8T1eZTIo/edit?usp=sharing) la présentation utilisée dans ce TP.

### Script de démarrage

```
DELETE FROM EMP WHERE ENAME IN ('Hichem','Mohamed');

Insert into EMP (EMPNO,ENAME,JOB,MGR,HIREDATE,SAL,COMM,DEPTNO) values ('7839','Mohamed','PLEASE',null,to_date('17/11/81','DD/MM/RR'),'2000',null,'10');
Insert into EMP (EMPNO,ENAME,JOB,MGR,HIREDATE,SAL,COMM,DEPTNO) values ('7698','Hichem','CRAFTSMAN','7839',to_date('01/05/81','DD/MM/RR'),'2800',null,'10');
```

## Introduction

Après avoir présenté les transactions de façon générale, nous allons présenter dans ce chapitre la gestion de la conccurence entre les transactions.

<p align="center">
  <img width="650" src="images/1.gif" alt="picture">
</p>

Tout d'abord nous présenterons les anomalies causées par cette concurrence puis nous présenterons les différents types d'isolation pour y remédier.

## Concurrence : Anomalies des Transactions

Il y a 3 types d'anomalies : 

### 1/ Mises à jour perdues

Elle se produit lorsque **deux transactions lisent une même donnée et la modifient, l’une après l’autre => une des écritures est perdues** .

Prenons par exemple 2 utilisateurs (transactions) qui veulent réserver un nombre de place pour un spectacle : 

<p align="center">
  <img width="600" src="images/2.png" alt="picture">
</p>


Lorsque ces 2 transactions lisent en même temps le nombre de place disponible les 2 vont lire **10 places**.
Ensuite chacune va réserver un nombre de place.
Une va réserver **2 places** et l'autre **4 places** .
Après que ces transactions soient exécutées, le nombre de place restant est de **6** hors que ca devrait être **4 places**.

### 2/ Lectures non répétables

Elle se produit lorsqu' **une transaction lit une même donnée 2 fois et nous constatons que la deuxième lecture est différente de la première**. 

Nous continuons avec le même exemple précédent :  

<p align="center">
  <img width="600" src="images/3.png" alt="picture">
</p>


Lorsqu'une transaction consulte le nombre de place disponible et qu'entre temps une autre transaction réserve **2 places**.
Tandis qu'après un traitement, la première transaction consulte à nouveau le nombre de place celui-ci s'avère modifié.


### 3/ Lectures sales

Elle se produit lorsqu' **il y a un entrelacement des transactions qui empêchent la bonne exécutions des Commit/Rollback**. 

Nous continuons avec le même exemple :  

<p align="center">
  <img width="600" src="images/4.png" alt="picture">
</p>


Lorsque 2 transactions consultent le nombre de place disponible et que l'une d'entre elle réserve et fait un **Commit** tandis que l'autre après un traitement décide de réserver des places puis fait un **Rollback**.
Le dernier **Rollback** ne s'exécutera pas correctement.

### 4/ Interblocages

Biensûr, lorsqu'on parle de gestion de conccurence entre plusieurs transactions, il est possible qu'un interblocage se produise.

<p align="center">
  <img width="600" src="images/5.png" alt="picture">
</p>

### Demo Interblocages :

| Timing | Session N° 1 (User1)   | Session N° 2 (User2) |Résultat | 
| :----: | :----: |:----:|:----:|
| t0 | ``` SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem');``` |||
| t1 | ``` UPDATE EMP SET SAL = 4000 WHERE ENAME ='Hichem'; ``` |------|------|
| t2 | ------ |```UPDATE EMP SET SAL = SAL + 1000 WHERE ENAME ='Mohamed';```|------|
| t3 | ```UPDATE EMP SET SAL = SAL + 1000 WHERE ENAME ='Mohamed';```|------|
| t4 | ------ |```UPDATE EMP SET SAL = SAL + 1000 WHERE ENAME ='Hichem';```|La session 1 va detecter l'interblocage |
| t5 | ```Commit;``` |------| Session 2: --> 1 row updated.|
| t6  |```UPDATE EMP SET SAL = SAL + 1000 WHERE ENAME ='Mohamed';```| ------|------|
| t7 |  ------ |```Commit;```| --------|
| t8 | ------ |```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem', 'Maaoui');```|------|

## Concurrence : Niveaux d'isolation des transactions

Plus le niveau est permissif, plus l’exécution est fluide, plus les anomalies sont possibles.
Plus le niveau est strict, plus l’exécution risque de rencontrer des blocages, moins les anomalies sont possibles.
Le niveau d’isolation par défaut n’est jamais le plus strict c'est pourquoi quand l’isolation totale est nécessaire, il faut l’indiquer explicitement.

Les 4 niveaux existants dans Oracle sont  **(ordonnés du moins strict au plus strict)** : 

- **Read uncommited** : tout est permis 

- **Read commited** : une requête accède à l’état de la base de données *au moment où la requête est exécutée*

- **Repeatable read** : Une requête accède à l’état de la base de données *au moment où la transaction a débutée*

- **Serializable** : Garantit une isolation totale => Cohérence de la base.

Parfois, le niveau le plus strict rejette des transactions voire provoquer des interblocages.
C'est pourquoi Oracle munit les développeurs de la clause ***FOR UPDATE***.
Autrement dit, le développeur déclare qu’une lecture va être suivie d’une mise à jour et le système pose un verrou fort pour celle-là, pas de verrou pour les autres et ainsi ca permet de monter le niveau de blocage uniquement quand c’est nécessaire... mais ca repose sur le facteur humain qui n'est pas assez fiable.

### Demo Niveau d'isolation  READ COMMITTED 

| Timing | Session N° 1  | Session N° 2 |Résultat | 
| :----: | :----: |:----:|:----:|
| t0| ``` SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem');``` |||
| t1 | ``` UPDATE EMP SET SAL = 4000 WHERE ENAME ='Hichem'; ``` |------|Updated|
| t2 | ------ |```SET TRANSACTION ISOLATION LEVEL READ COMMITTED;```|Transaction set|
| t3 | ------ |```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem');```|la requete de User 1 n'est pas effectuée, pas de "commit" alors "verrouillé"
| t4 | ------ |```UPDATE EMP SET SAL = 3800 WHERE ENAME ='Mohamed';```|Updated|
| t5 | ```Insert into EMP (EMPNO,ENAME,JOB,MGR,HIREDATE,COMM,DEPTNO) values ('9999','Maaoui','Magician',null,to_date('17/02/2021','DD/MM/RR'),null,'10');``` |------|Row created|
| t6 | ------ |```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem', 'Maaoui');```|inserted row isn't shown|
| t7 | ------ |```UPDATE EMP SET SAL = 5000 WHERE ENAME ='Hichem';```|Blocked|
| t8 | ```Commit;``` |------|la requete precedente de User 2 est effectuée|
| t9 | ------ |```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem', 'Maaoui');```|displays the previous update + the previously inserted row by User1|
| t10| ------ |```COMMIT;```|Commited|
| t11| ```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem', 'Maaoui');```|------|changes by User2 are now also visible to User1|




### Demo Niveau d'isolation SERIALIZABLE ;

| Timing | Session N° 1  | Session N° 2 |Résultat | 
| :----: | :----: |:----:|:----:|
| t0| ``` SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem');``` |||
| t1| ``` UPDATE EMP SET SAL = 4000 WHERE ENAME ='Hichem'; ``` |------|Updated|
| t2| ------ |```SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;```|Transaction set|
| t3| ------ |```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem');```|requete de User 1 n'est pas effectuée "verrouillé"|
| t4| ------ |```UPDATE EMP SET SAL = 3800 WHERE ENAME ='Mohamed';```|Updated|
| t5| ```Insert into EMP (EMPNO,ENAME,JOB,MGR,HIREDATE,COMM,DEPTNO) values ('9999','Maaoui','Magician',null,to_date('17/02/2021','DD/MM/RR'),null,'10');``` |------|Row created|
| t6| ```COMMIT;```|------ |commited|
| t7|```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem', 'Maaoui');```| ------ |displays the changes made by User 1|
| t8| ------ |```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem', 'Maaoui');```|displays the changes made only by User 2 despite the previous commit from User 1|
| t9| ```Commit;``` |------|commited|
| t10|```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem', 'Maaoui');```| ------ |displays the changes made by both users|
| t11| ------ |```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem', 'Maaoui');```|displays the changes made by both users|
| t12| ------ | ```COMMIT;```|------|
| t13| ``` UPDATE EMP SET SAL = 5000 WHERE ENAME ='Maaoui'; ``` |------|updated|
| t14| ------ |```SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;```|Transaction set|
| t15| ------ |```UPDATE EMP SET SAL = 5200 WHERE ENAME ='Maaoui';```|blocked|
| t16| ```COMMIT;``` |------|commit complete, can't serialize access for previous transaction|
| t17| ------ |```ROLLBACK;```|rollback complete|
| t18| ------ |```SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;```|Transaction set|
| t19| ------ |```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem', 'Maaoui');```|Update by User 1 is displayed|
| t20| ``` UPDATE EMP SET SAL = 5200 WHERE ENAME ='Maaoui'; ``` |------|Updated|
| t21| ```COMMIT;``` |------|commited|
| t22| ------ | ```COMMIT;```|commited|
| t23| ------ |```SELECT ENAME, SAL FROM EMP WHERE ENAME IN ('Mohamed','Hichem', 'Maaoui');```|previous update by User 1 is displayed|



### Exemple de traitement avec "FOR UPDATE" ;

```sh
SET SERVEROUTPUT ON ;

DECLARE 

tmp_v_empno number (6,2);
CURSOR cur IS SELECT EMPNO FROM EMP WHERE ENAME = 'Mohamed' FOR UPDATE of SAL;

BEGIN 
   DBMS_OUTPUT.PUT_LINE('START');

   OPEN cur;   

    FETCH cur INTO tmp_v_empno;

      IF cur%notfound THEN
          tmp_v_empno := -1 ; 
          DBMS_OUTPUT.PUT_LINE('This Employee is currently being Updated, try again in a while');
      ELSE
          DBMS_OUTPUT.PUT_LINE(' Updating Employee Number  ' || tmp_v_empno );
          UPDATE EMP SET SAL = 6666 WHERE CURRENT OF cur ;
          COMMIT;
    END IF;

   CLOSE cur;

   DBMS_OUTPUT.PUT_LINE(' Employee Number  ' || tmp_v_empno || ' Updated successfully !!'  );

END;
/

```

Pour vérifier :
```sh
SELECT EMPNO, SAL FROM EMP WHERE ENAME = 'Mohamed' ;
```
## Conclusion

- Comprendre et utiliser correctement les niveaux d’isolation est impératif pour les applications transactionnelles

- Savoir repérer les transactions dans une application. Elle doivent respecter lacohérence applicative(le système ne peut pas la deviner pour vous)

- La clausefor updateest en théorie meilleure, mais repose sur le facteur humain

- Savoir qu’en mode sérialisable on risque des rejets

- Comprendre les risques dans les autres modes.
