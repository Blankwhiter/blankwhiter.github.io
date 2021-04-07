# Vue中使用fullcalendar
1.安装插件
yarn add @fullcalendar/vue @fullcalendar/core @fullcalendar/daygrid @fullcalendar/interaction @fullcalendar/list @fullcalendar/timegrid
2.引入组件
 ```
<template>
	<view class="content">
		<FullCalendar ref="fullcalendar"   :options='calendarOptions'>
		</FullCalendar>
	</view>
</template>

<script>
	import FullCalendar from '@fullcalendar/vue'
	import dayGridPlugin from '@fullcalendar/daygrid'
	import timeGridPlugin from '@fullcalendar/timegrid'
	import listPlugin from '@fullcalendar/list'
	import interactionPlugin from '@fullcalendar/interaction'
	export default {
		data() {
			return {
				calendarOptions: {
					// 引入的插件，比如fullcalendar/daygrid，fullcalendar/timegrid引入后才可显示月，周，日
					plugins: [
						dayGridPlugin,
						timeGridPlugin,
						interactionPlugin,
						listPlugin  // needed for dateClick
					],
					// 切换语言，当前为中文
					locale: "zh-cn",
					height: 'auto',
					// 切换视图按钮
					buttonText: {
						today: '今天',
						month: '月',
						week: '周',
						day: '天'
					},
					// 设置一周中显示的第一天是哪天，周日是0，周一是1，类推
       				firstDay: 1, 
       				// 日历头部按钮位置
					headerToolbar: {
						left: 'prev,next today',
						center: 'title',
						right: 'dayGridMonth,timeGridWeek,timeGridDay'
					},
					// 默认为那个视图（月：dayGridMonth，周：timeGridWeek，日：timeGridDay）
					initialView: 'dayGridMonth',
					// timeGridEventMinHeight: '20', // 设置事件的最小高度
       			    // aspectRatio: 1.65, // 设置日历单元格宽度与高度的比例。
       			    //eventLimit: true, // 设置月日程，与all-day slot的最大显示数量，超过的通过弹窗显示
					initialEvents: [{
							id: 1,
							title: 'All-day event',
							start: new Date().toISOString().replace(/T.*$/, '')
						},
						{
							id: 2,
							title: 'Timed event',
							start: new Date().toISOString().replace(/T.*$/, '') + 'T12:00:00'
						}
					], // alternatively, use the `events` setting to fetch from a feed
					// 事件
			        eventClick: this.handleEventClick, // 点击日历日程事件
			        eventDblClick: this.handleEventDblClick, // 双击日历日程事件 (这部分是在源码中添加的)
			        eventClickDelete: this.eventClickDelete, // 点击删除标签事件 (这部分是在源码中添加的)
			        eventDrop: this.eventDrop, // 拖动日历日程事件
			        eventResize: this.eventResize, // 修改日历日程大小事件
			        select: this.handleDateSelect, // 选中日历格事件
			        eventDidMount: this.eventDidMount, // 安装提示事件
			        // loading: this.loadingEvent, // 视图数据加载中、加载完成触发（用于配合显示/隐藏加载指示器。）
			        // selectAllow: this.selectAllow, //编程控制用户可以选择的地方，返回true则表示可选择，false表示不可选择
			        eventMouseEnter: this.eventMouseEnter, // 鼠标滑过
			        allowContextMenu: false,
			        editable: true, // 是否可以进行（拖动、缩放）修改
			        eventStartEditable: true, // Event日程开始时间可以改变，默认true，如果是false其实就是指日程块不能随意拖动，只能上下拉伸改变他的endTime
			        eventDurationEditable: true, // Event日程的开始结束时间距离是否可以改变，默认true，如果是false则表示开始结束时间范围不能拉伸，只能拖拽
			        selectable: true, // 是否可以选中日历格
			        selectMirror: true,
			        selectMinDistance: 0, // 选中日历格的最小距离
			        dayMaxEvents: true,
			        weekends: true,
			        navLinks: true, //  链接
			        selectHelper: false,
			        slotEventOverlap: false // 相同时间段的多个日程视觉上是否允许重叠，默认true允许
					eventsSet: this.handleEvents
					/* you can update a remote database when these fire:
					eventAdd:
					eventChange:
					eventRemove:
					*/
				},
				currentEvents: [],
				i:3,
			}
		},
		components: {
			FullCalendar
		},
		onLoad() {

		},
		methods: {
			handleWeekendsToggle() {
				this.calendarOptions.weekends = !this.calendarOptions.weekends // update a property
			},
			handleDateSelect(selectInfo) {
				alert(1)
				let title = prompt('Please enter a new title for your event')
				let calendarApi = selectInfo.view.calendar
				calendarApi.unselect() // clear date selection
				if (title) {
					calendarApi.addEvent({
						id: this.i++,
						title,
						start: selectInfo.startStr,
						end: selectInfo.endStr,
						allDay: selectInfo.allDay
					})
				}
			},
			handleEventClick(clickInfo) {
				if (confirm(`Are you sure you want to delete the event '${clickInfo.event.title}'`)) {
					clickInfo.event.remove()
				}
			},
			handleEvents(events) {
				this.currentEvents = events
			}
		}
	}
</script>

<style>

</style>

```


附录：
在mounted中添加获取日历信息方法
```
mounted() {
    this.calendarApi = this.$refs.fullCalendar.getApi();
}
```