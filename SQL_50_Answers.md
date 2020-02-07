### --1 查询“ 01 ”课程比" 02 "课程成绩高的学生的信息及课程分数
~~~~sql
SELECT A.SId, A.score as score1, B.score as score2
FROM (SELECT * FROM SC WHERE CId = '01') as A 
INNER JOIN (SELECT * FROM SC WHERE CId = '02') as B
ON A.SId = B.SId 
WHERE A.score > B.score;
~~~~

#### --1.1 查询同时存在" 01 "课程和" 02 "课程的情况
##### It's a bit unclear what output the question asks; but you get the idea
~~~~sql
SELECT A.SId as SId
FROM (SELECT * FROM SC WHERE CId = '01') as A 
INNER JOIN (SELECT * FROM SC WHERE CId = '02') as B
ON A.SId = B.SId;
~~~~

### --2. 查询平均成绩大于等于 60 分的同学的学生编号和学生姓名和平均成绩
~~~~sql
SELECT t1.SId, s.Sname,round(t1.AVGScore, 1)
FROM (SELECT SId, AVG(SCORE) as AVGScore
FROM SC
GROUP BY SID
HAVING AVG(SCORE) >= 60) as t1 
INNER JOIN Student s
ON s.SId = t1.SId
~~~~

### --4. 查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩(没成绩的显示为null)
~~~~sql
SELECT S.SId, S.Sname, t1.totalnumcourses, t1.totalscores
FROM Student S
LEFT JOIN (SELECT SId, COUNT(CId) as totalnumcourses, SUM(score) as totalscores
FROM SC
GROUP BY SId) t1
ON S.SId = t1.SId
~~~~
#### --4.1 查有成绩的学生信息
~~~~sql
SELECT S.SId, S.Sname, t1.totalnumcourses, t1.totalscores
FROM Student S
INNER JOIN (SELECT SId, COUNT(CId) as totalnumcourses, SUM(score) as totalscores
FROM SC
GROUP BY SId) t1
ON S.SId = t1.SId	
~~~~
### --5.查询「李」姓老师的数量
```{SQL}
SELECT COUNT(Tname)
FROM Teacher
WHERE Tname LIKE '李%'
```

### --6.查询学过「张三」老师授课的同学的信息
~~~~sql
SELECT *
FROM Student
WHERE SId in (SELECT SId FROM SC WHERE CId in (SELECT CId FROM Course WHERE TId in (SELECT TId FROM Teacher WHERE Tname = '张三')))
~~~~

### --7.查询没有学全所有课程的同学的信息
~~~~sql
SELECT * FROM Student
WHERE SId not in
(SELECT SId FROM SC
GROUP BY SId
HAVING COUNT(CId) =(SELECT COUNT(CId) FROM Course));
~~~~

### --8. 查询至少有一门课与学号为" 01 "的同学所学相同的同学的信息 
~~~~sql
SELECT * FROM Student
WHERE SId in (SELECT DISTINCT SId FROM SC
WHERE CId in (SELECT DISTINCT CId FROM SC WHERE SId = '01') and SId !='01')
~~~~

### --9. 查询和" 01 "号的同学学习的课程完全相同的其他同学的信息 
~~~~sql
SELECT * FROM Student
WHERE SId in (SELECT DISTINCT SId FROM SC
WHERE CId in (SELECT DISTINCT CId FROM SC WHERE SId = '01') and SId !='01'
GROUP BY SId
HAVING COUNT(CId) = (SELECT COUNT(DISTINCT CId) FROM SC WHERE SId = '01'))
~~~~

### --10. 查询没学过「张三」老师讲授的任一门课程的学生姓名 
~~~~sql
SELECT *
FROM Student
WHERE SId not in (SELECT SId FROM SC WHERE CId in (SELECT CId FROM Course WHERE TId in (SELECT TId FROM Teacher WHERE Tname = '张三')))
~~~~

### --11.查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩
~~~~sql
SELECT s.SId, s.Sname, round(t1.avgscore)
FROM Student s
RIGHT JOIN (SELECT SId, SUM(CASE WHEN score <60 THEN 1 ELSE 0 END) as failnum, AVG(score) as avgscore
FROM SC GROUP BY SId) t1
ON s.SId = t1.SId
WHERE t1.failnum >=2
~~~~

~~~~sql

SELECT s.SId, s.Sname, round(t1.avgscore)
FROM Student s
RIGHT JOIN (SELECT SId, AVG(score) as avgscore
FROM SC WHERE score <60 GROUP BY SId HAVING count(score >=2)) t1
ON s.SId = t1.SId
~~~~

### --12.检索" 01 "课程分数小于 60 ，按分数降序排列的学生信息
~~~~sql
SELECT * FROM Student S
RIGHT JOIN (SELECT SId,score FROM SC
WHERE CId = '01' and score < 60 ORDER BY score DESC) t1
ON S.SId = t1.SId
~~~~

### --13. 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩
~~~~sql
SELECT S.SId, ifnull(t1.score01,0),ifnull(t1.score02,0),ifnull(t1.score03,0), round(ifnull(t1.avgscore,0),1)
FROM (SELECT SId,MAX(CASE WHEN CId= '01' THEN score ELSE 0 END) score01,
MAX(CASE WHEN CId= '02' THEN score ELSE 0 END) score02,
MAX(CASE WHEN CId= '03' THEN score ELSE 0 END) score03, AVG(score) avgscore from SC
GROUP BY SId
ORDER BY AVG(score) DESC) t1
RIGHT JOIN Student S
On S.SId = t1.SId
~~~~
### 14.查询各科成绩最高分、最低分和平均分:
### --以如下形式显示：课程 ID ，课程 name ，最高分，最低分，平均分，及格率，中等率，优良率，优秀率
### --及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90
~~~~sql
SELECT C.Cname, t1.*
FROM (SELECT CId, MAX(score) as maxscore,MIN(score) as minscore, ROUND(AVG(score),1) as avgscore, 
ROUND(SUM(CASE WHEN score >=60 THEN 1 ELSE 0 END)/COUNT(*)*100,1) as Passed,
ROUND(SUM(CASE WHEN score >=70 and score <80 THEN 1 ELSE 0 END)/COUNT(*)*100,1) as C,
ROUND(SUM(CASE WHEN score >=80 and score <90 THEN 1 ELSE 0 END)/COUNT(*)*100,1) as B,
ROUND(SUM(CASE WHEN score >=90 THEN 1 ELSE 0 END)/COUNT(*)*100,1) as A
FROM SC GROUP BY CId) t1
LEFT JOIN Course C
ON C.CId = t1.CId
~~~~
