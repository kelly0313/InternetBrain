# SQL练习答案

- 创建学生表

CREATE TABLE Student(
	Sid CHAR(9) PRIMARY KEY,
	Sname CHAR(20),
	Sage SMALLINT,
	Ssex CHAR(2)
)

- 创建课程表

CREATE TABLE Course(
	Cid CHAR(9) PRIMARY KEY,
	Cname CHAR(20),
	Tid CHAR(9)
)

- 创建成绩表

CREATE TABLE SC(
	Sid CHAR(9),
	Cid CHAR(9),
	score SMALLINT,
  PRIMARY KEY (Sid,Cid)
)

- 创建教师表

CREATE TABLE Teacher(
	Tid CHAR(9) PRIMARY KEY,
	Tname CHAR(20)
)
***

Student(Sid,Sname,Sage,Ssex) 学生表
Course(Cid,Cname,Tid) 课程表
SC(Sid,Cid,score) 成绩表
Teacher(Tid,Tname) 教师表
***
练习内容：
1. 查询“某1”课程比“某2”课程成绩高的所有学生的学号；
SELECT a.sid FROM (SELECT sid,score FROM SC WHERE cid=1) a,(SELECT sid,score FROM SC WHERE cid=3) b WHERE a.score>b.score AND a.sid=b.sid;
此题知识点，嵌套查询和给查出来的表起别名
2. 查询平均成绩大于60分的同学的学号和平均成绩；
SELECT sid,avg(score)  FROM sc  GROUP BY sid having avg(score) >60;
此题知识点，GROUP BY 语句用于结合合计函数，根据一个或多个列对结果集进行分组。group by后面不能接where，having代替了where
3. 查询所有同学的学号、姓名、选课数、总成绩
SELECT Student.sid,Student.Sname,count(SC.cid),sum(score)FROM Student left Outer JOIN SC on Student.sid=SC.cid GROUP BY Student.sid,Sname
4. 查询姓“李”的老师的个数；
select count(teacher.tid)from teacher where teacher.tname like'李%'
5. 查询没学过“叶平”老师课的同学的学号、姓名；
SELECT Student.sid,Student.Sname FROM Student WHERE sid not in (SELECT distinct( SC.sid) FROM SC,Course,Teacher WHERE  SC.cid=Course.cid AND Teacher.id=Course.tid AND Teacher.Tname='叶平');
 此题知识点，distinct是去重的作用
6. 查询学过“```”并且也学过编号“```”课程的同学的学号、姓名；
select a.SID,a.SNAME from (select student.SNAME,student.SID from student,course,sc where cname='c++'and sc.sid=student.sid and sc.cid=course.cid) a,
(select student.SNAME,student.SID from student,course,sc where cname='english'and sc.sid=student.sid and sc.cid=course.cid) b where a.sid=b.sid;
标准答案（但是好像不好使）SELECT Student.S#,Student.Sname FROM Student,SC WHERE Student.S#=SC.S# AND SC.C#='001'and exists( SELECT * FROM SC as SC_2 WHERE SC_2.S#=SC.S# AND SC_2.C#='002');  
此题知识点，exists是在集合里找数据，as就是起别名
7. 查询学过“叶平”老师所教的所有课的同学的学号、姓名；
select a.sid,a.sname from (select student.sid,student.sname from student,teacher,course,sc 
where teacher.TNAME='杨巍巍' and teacher.tid=course.tid and course.cid=sc.cid and student.sid=sc.sid) a
标准答案：SELECT sid,Sname FROM Student WHERE sid in (SELECT sid FROM SC ,Course ,Teacher WHERE SC.cid=Course.cid AND Teacher.tid=Course.tid AND Teacher.Tname='杨巍巍' GROUP BY sid having count(SC.cid)=(SELECT count(cid) FROM Course,Teacher  WHERE Teacher.tid=Course.tid AND Tname='杨巍巍'))
8. 查询课程编号“”的成绩比课程编号“”课程低的所有同学的学号、姓名；
select a.sid,a.sname from(select student.SID,student.sname,sc.SCORE  from student,sc where student.sid=sc.sid and sc.cid=1) a,
(select student.SID,student.sname,sc.score from student,sc where student.sid=sc.sid and sc.cid=2) b where a.score<b.score and a.sid=b.sid
标准答案：SELECT sid,Sname FROM (SELECT Student.sid,Student.Sname,score ,
(SELECT score FROM SC SC_2 WHERE SC_2.sid=Student.sid AND SC_2.cid=1) score2 FROM Student,SC
WHERE Student.sid=SC.sid AND cid=1) S_2 WHERE score2 <score;
9. 查询所有课程成绩小于分的同学的学号、姓名；
SELECT sid,Sname FROM Student WHERE sid not in (SELECT Student.sid FROM Student,SC WHERE Student.sid=SC.sid AND score>60); 
此题知识点，先查出大于60分的，然后not in 就是小于60分的了
10. 查询没有学全所有课的同学的学号、姓名；
SELECT Student.sid,Student.Sname  FROM Student,SC  
WHERE Student.sid=SC.sid GROUP BY  Student.sid,Student.Sname having count(cid) <(SELECT count(cid) FROM Course); 
11. 查询至少有一门课与学号为“”的同学所学相同的同学的学号和姓名；
12. 查询至少学过学号为“”同学所有一门课的其他同学学号和姓名；
SELECT student.sid,student.Sname FROM Student,SC WHERE Student.sid=SC.sid AND cid in (SELECT cid FROM SC WHERE sid=1)
此题知识点，SELECT sid,Sname FROM Student,SC WHERE Student.sid=SC.sid AND cid in (SELECT cid FROM SC WHERE sid=1)这样写是错误的，因为from后面是两个表，不能明确是哪个表里面的sid和sname所以错误提示是“未明确定义列”
13. 把“SC”表中“叶平”老师教的课的成绩都更改为此课程的平均成绩；
update sc set score=(select avg(score) from sc,course,teacher where course.cid=sc.cid and course.tid=teacher.tid and teacher.tname='杨巍巍')
14. 查询和“”号的同学学习的课程完全相同的其他同学学号和姓名；
SELECT sid FROM SC WHERE cid in (SELECT cid FROM SC WHERE sid=6) GROUP BY sid having count(*)=(SELECT count(*) FROM SC WHERE sid=6); 
此题知识点，用数量来判断 
15. 删除学习“叶平”老师课的SC表记录； 
delete from sc s where s.cid in (select c.cid from teacher t,course c where t.tid = c.tid and tname='李子')
此题知识点，嵌套查询可以分布考虑，先查出李子老师都交了什么课的id，然后再删除那些id的值
16. 向SC表中插入一些记录，这些记录要求符合以下条件：没有上过编号“”课程的同学学号、课程的平均成绩；
Insert into SC SELECT sid,2,(SELECT avg(score) FROM SC WHERE cid=2) FROM Student WHERE sid not in (SELECT sid FROM SC WHERE cid=2); 
17. 按平均成绩从高到低显示所有学生的“数据库”、“企业管理”、“英语”三门的课程成绩，按如下形式显示：学生ID,,数据库,企业管理,英语,有效课程数,有效平均分；(没做出来)
18. 查询各科成绩最高和最低的分：以如下形式显示：课程ID，最高分，最低分；
select cid as 课程号,max(score)as 最高分,min(score) as 最低分 from sc group by cid
标准答案（但是运行不好使）SELECT L.cid As 课程ID,L.score AS 最高分,R.score AS 最低分 
FROM SC L ,SC AS R  
WHERE L.cid = R.cid AND  
L.score = (SELECT MAX(IL.score)  
FROM SC AS IL,Student AS IM  
WHERE L.cid = IL.cid AND IM.sid=IL.sid  
GROUP BY IL.cid)  
AND  R.Score = (SELECT MIN(IR.score) FROM SC AS IR WHERE R.cid = IR.cid  GROUP BY IR.cid ); 
19. 按各科平均成绩从低到高和及格率的百分数从高到低顺序
26. 查询每门课程被选修的学生数
select sc.cid,count(sc.sid) from sc,course where sc.cid=course.cid group by sc.cid
27. 查询出只选修了一门课程的全部学生的学号和姓名
SELECT SC.sid,Student.Sname,count(cid) AS 选课数 FROM SC ,Student  
WHERE SC.sid=Student.sid GROUP BY SC.sid ,Student.Sname having count(cid)=1;
32. 查询每门课程的平均成绩，结果按平均成绩升序排列，平均成绩相同时，按课程号降序排列
SELECT Cid,Avg(score) FROM SC GROUP BY cid ORDER BY Avg(score),cid DESC ;
37. 查询不及格的课程，并按课程号从大到小排列 
SELECT cid,sid FROM sc WHERE score <60 ORDER BY cid 
38. 查询课程编号为且课程成绩在分以上的学生的学号和姓名；
select student.sid,student.sname from sc,student where sc.cid=1 and sc.score>60 and sc.sid=student.sid
40. 查询选修“叶平”老师所授课程的学生中，成绩最高的学生姓名及其成绩
select student.sname,sc.score from sc,student,teacher,course c where teacher.tname='李子'
and teacher.tid=c.tid and c.cid=sc.cid and sc.sid=student.sid and sc.score=(select max(score)from sc where sc.cid=c.cid)
41. 查询各个课程及相应的选修人数
select sc.cid ,count(sc.sid)from sc,student where sc.sid=student.sid group by sc.cid 
43. 查询每门功成绩最好的前两名

44.统计每门课程的学生选修人数（超过人的课程才统计）。要求输出课程号和选修人数，查询结果按人数降序排列，查询结果按人数降序排列，若人数相同，按课程号升序排列
select sc.cid,count(sc.cid)from sc,course where sc.cid=course.cid group by sc.cid  order by sc.cid desc
45.检索至少选修两门课程的学生学号
SELECT sid FROM  sc group  by  sid having  count(*)  >  =  2  

rownum的用法
查询所有成绩第二名到第四名的成绩
select * from （select rownum p,t.score from（SELECT s.score score FROM sc s ORDER BY score desc）t ）tt where tt.p>1 and tt.p<5

47.查询没学过“叶平”老师讲授的任一门课程的学生姓名
select distinct sid from sc where sid not in(select sc.sid from sc,course,teacher where sc.cid=course.cid and course.tid=teacher.tid and 
teacher.tname='杨巍巍')
48.查询两门以上不及格课程的同学的学号及其平均成绩
49.检索“”课程分数小于，按分数降序排列的同学学号
select sc.sid from sc,course where sc.cid=course.cid and course.cname='java' and sc.score<90
50.删除“”同学的“”课程的成绩 
delete from sc where sid=1 and cid=1