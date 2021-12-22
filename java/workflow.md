

```markdown
# RE 表示 repository，这个前缀的表包含了流程定义和流程静态资源(图片，规则，等等)
act_RE_* 
# RU 表示 runtime，这些运行时的表，包含流程实例、任务、变量、异步任务等运行中的数据。只在流程实例执行过程中保存这些数据，在流程结束时就会删除这些记录，这样来尽量使运行时表可以一直很小速度很快
act_RU_*
# HI 表示 history，这些表包含历史数据，比如历史流程实例、遍历、任务等等
act_HI_* 
# GE 表示 general，通用数据，用于不同场景下
act_GE_* 


# 资源（二进制数据）表
act_ge_bytearray 
# 流程部署表
act_re_deployment 
# 流程定义表
act_re_procdef 
# 运行时流程实例(执行流)表
act_ru_execution 
# 流程与身份关系表(执行流)
act_ru_identitylink 
# 运行时流程任务(节点)表(执行流)
act_ru_task 
# 运行时流程参数(变量)表(执行流)
act_ru_variable 
```





![image-20211222105332820](/Users/admin/WorkSpace/github/prc/note/java/workflow.assets/image-20211222105332820.png)