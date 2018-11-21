<template>
  <div class="home">
    <van-cell-group>
      <van-field
        v-model="formatStartTime"
        :readonly="true"
        placeholder="选取开始时间（日/时/分）"
        @click="isShowStartTime = !isShowStartTime"
      >
        <van-checkbox slot="button" v-model="newEvent.all_day" shape="square">全天</van-checkbox>
      </van-field>
      <van-field
        v-model="formatEndTime"
        :readonly="true"
        placeholder="选取结束时间（日/时/分）"
        @click="isShowEndTime = !isShowEndTime"
      />
    </van-cell-group>
    <!-- 开始时间选择器 -->
    <van-popup v-model="isShowStartTime" position="bottom">
      <van-picker
        :columns="timeColumns(7)"
        show-toolbar
        title="选择开始时间"
        @confirm="chooseStartTime"
        @cancel="closeStartTimePicker"
        @change="addYear"
      />
    </van-popup>
    <!-- 结束时间选择器 -->
    <van-popup v-model="isShowEndTime" position="bottom">
      <van-picker
        :columns="timeColumns(8)"
        show-toolbar
        title="选择结束时间"
        @confirm="chooseEndTime"
        @cancel="closeEndTimePicker"
        @change="addYear"
      />
    </van-popup>
  </div>
</template>

<script>
import moment from 'moment';

export default {
  name: 'home',
  data() {
    return {
      time: [],
      endTime: '',
      startTime: '',
      isShowCreateEvent: false,
      isShowType: false,
      isShowStartTime: false,
      isShowEndTime: false,
      isLoading: false,
      lunarDate: '',
      selectDate: '',
      events: [],
      newEvent: {
        title: '',
        start_time: '',
        end_time: '',
        type: '',
        description: '',
        all_day: false,
        location: '',
      },
      columns: ['福州', '厦门', '莆田', '三明', '泉州'],
      years: [],
      momentDays: [],
      defaultIndex: '',
    };
  },
  mounted() {
    // 设置选择器的默认年份
    this.years.push(moment().get('year'));

    // 设置选择器的默认下标
    this.defaultIndex = Number(moment().dayOfYear() - 1);
  },
  computed: {
    formatStartTime() {
      if (!this.startTime) return '';
      const startTime = moment(this.startTime);

      const formatStartTime = this.newEvent.all_day
        ? `${startTime.format('MM/DD')}`
        : `${startTime.format('MM/DD HH:mm')}`;

      return formatStartTime;
    },
    formatEndTime() {
      if (!this.endTime) return '';
      const endTime = moment(this.endTime);

      const formatEndTime = this.newEvent.all_day
        ? `${endTime.format('MM/DD')}`
        : `${endTime.format('MM/DD HH:mm')}`;

      return formatEndTime;
    },
  },
  methods: {
    // 时间选择器可选范围
    timeColumns(defaultHour) {
      const dateTime = {
        year: [],
        month: [],
        day: [],
        hour: [],
        minute: [],
      };

      // 获取100年的年份数据 1970 - 2070
      dateTime.year = dateTime.year.fill
        .call(new Array(100), 0)
        .map((item, index) => `${index + 1970}`);

      // 获取全年的月份数据
      dateTime.month = dateTime.month.fill
        .call(new Array(12), 0)
        .map((item, index) => `${index + 1}`);


      // 设置全年的天数数据
      const momentDays = [];
      // 计算数据
      this.years.forEach((year) => {
        dateTime.month.forEach((month) => {
          const formatDaysArray = [];
          const daysArray = [];
          // 生成全年的格式化数据用于展示
          const formatDaysOfYear = formatDaysArray.fill
            .call(
              new Array(moment(`${year}-${month}`, 'YYYY-M').daysInMonth()),
              0,
            )
            .map((item, index) => {
              // 当天的数据显示为 今天
              if (moment().isSame(`${year}-${month}-${index + 1}`, 'day')) {
                return '今天';
              }

              return moment(`${year}-${month}-${index + 1}`).locale('zh-cn').format('MM月DD日 ddd');
            });

          // 生成全年的 moment() 用于获取数据
          const daysOfYear = daysArray.fill
            .call(
              new Array(moment(`${year}-${month}`, 'YYYY-M').daysInMonth()),
              0,
            )
            .map((item, index) => moment(`${year}-${month}-${index + 1}`));

          // 存入数据
          dateTime.day.push(...formatDaysOfYear);
          momentDays.push(...daysOfYear);
        });
      });

      this.momentDays = momentDays;

      // 设置一天的小时数
      dateTime.hour = dateTime.hour.fill
        .call(new Array(24), 0)
        .map((item, index) => index + 1);

      // 获取一小时的分钟数
      const minuteArray = dateTime.hour.fill
        .call(new Array(60), 0)
        .map((item, index) => index);

      // 过滤，设置分钟只显示 5 的倍数
      dateTime.minute = minuteArray.filter(minute => minute % 5 === 0);

      // 通过全天是否勾选设置是日期选择器还是时间选择器
      const timeColumns = this.newEvent.all_day
        ? [
          {
            values: dateTime.year.map(year => `${year}年`),
            className: 'column1',
            defaultIndex: dateTime.year.indexOf(moment().get('year').toString()),
          },
          {
            values: dateTime.month.map(month => `${month}月`),
            className: 'column2',
            defaultIndex: moment().get('month'),
          },
        ]
        : [
          {
            values: dateTime.day,
            className: 'column1',
            defaultIndex: this.defaultIndex,
          },
          {
            values: dateTime.hour,
            className: 'column2',
            defaultIndex: defaultHour,
          },
          {
            values: dateTime.minute,
            className: 'column3',
          },
        ];

      return timeColumns;
    },
    // 选择开始时间
    chooseStartTime(value, index) {
      this.startTime = this.momentDays[index[0]].set({ hour: value[1], minute: value[2] });
      this.newEvent.start_time = this.startTime.toISOString();
      console.log(this.startTime);
      this.isShowStartTime = false;
    },
    // 关闭开始时间选择器
    closeStartTimePicker() {
      this.isShowStartTime = false;
    },
    // 选择结束时间
    chooseEndTime(value, index) {
      this.endTime = this.momentDays[index[0]].set({ hour: value[1], minute: value[2] });
      this.newEvent.end_time = this.endTime.toISOString();
      this.isShowEndTime = false;
    },
    // 关闭结束时间选择器
    closeEndTimePicker() {
      this.isShowEndTime = false;
    },
    // 动态添加年份
    addYear(picker) {
      // 获取当前滚动的日期下标
      const currentIndex = picker.getColumnIndex(0);

      // 如果向下滚动超过了当前 moment 数组的倒数第 60 个，则动态添加下一年的日期数据
      if (this.momentDays[currentIndex].isAfter(this.momentDays[this.momentDays.length - 60])) {
        // 当前年份数组最后一个年份
        const year = this.years[this.years.length - 1];
        // 重置默认下标
        this.defaultIndex = currentIndex;
        // 添加下一年的日期数据
        this.years.push(year + 1);

        if (this.years.length >= 3) {
          // 如果年份超过3年，则删除最开始一年的数据
          this.years.shift();
          // 重置默认下标，数值为当前日期的下标减去当年的天数（判断是否闰年）
          this.defaultIndex = currentIndex - (moment([year - 1]).isLeapYear() ? 366 : 365);
        }
      }

      // 如果向上滚动超过了当前 moment 数组的第 60 个，则动态添加上一年的日期数据
      if (this.momentDays[currentIndex].isBefore(this.momentDays[60])) {
        // 当前年份数组第一个年份的上一年
        const year = this.years[0] - 1;
        // 重置默认下标，数值为当前日期的下标加上当年的天数（判断是否闰年）
        this.defaultIndex = currentIndex + (moment([year]).isLeapYear() ? 366 : 365);
        // 添加上一年的日期数据
        this.years.unshift(year);

        if (this.years.length >= 3) {
          // 如果年份超过3年，则删除最后面一年的数据
          this.years.pop();
          // 重置默认下标，
          this.defaultIndex = currentIndex;
        }
      }
    },
  },
};
</script>
