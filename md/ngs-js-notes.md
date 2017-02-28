
浮屏打开一个动态页面
	title就是浮出来的标题
	message就是浮出来的消息内容
	event表示我要打开一个动态页面
	eventData里面写了具体要打开哪个动态页面，tab名字，其他参数，tab是否可以关闭
	urgency表示这个浮出来的消息是不是紧急的
	lastTime表示持续多久自己滑下去，单位毫秒



     window.H5Utils.popup({
     	title: '提示标题', 
     	message: '提示内容',
     	event: window.$CONST.EVENT.OPEN_DYN,
     	eventData: {
     		dynId : 'SB1234',
     		tabName : 'TAB名字',
     		parameter : {},
     		keepOpen : false
     	},
     	urgency: true,
     	lastTime: 5000
     });



浮屏打开一个工单
	oId表示要打开的工单的objective_guid
	otId表示要打开的工单的objectivetype_id
	subId表示要打开的工单的协办步骤guid
	这三个可以选一个传递就好

     window.H5Utils.popup({
     	title: '提示标题', 
     	message: '提示内容',
     	event: window.$CONST.EVENT.OPEN_WO,
     	eventData: {
     		addinId : 'ADDINWORDER',
     		tabName : 'TAB名字',
     		oId : '',
     		otId : '',
     		subId : ''
     	},
     	urgency: true,
     	lastTime: 5000
     });