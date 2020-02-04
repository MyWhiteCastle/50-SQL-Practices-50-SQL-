# --1 查询“ 01 ”课程比" 02 "课程成绩高的学生的信息及课程分数
```{python}
SELECT A.SId, A.score as score1, B.score as score2
FROM (SELECT * FROM SC WHERE CId = '01') as A 
INNER JOIN (SELECT * FROM SC WHERE CId = '02') as B
ON A.SId = B.SId 
WHERE A.score > B.score;
```
