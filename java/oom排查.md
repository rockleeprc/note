

	
	# 确认是不是内存本身就分配过小
	1. jmap -heap 10765
	# 最耗内存的对象
	2. jmap -histo:live 10765 | more 
	#
	3. /proc/${PID}/fd
		/proc/${PID}/task
