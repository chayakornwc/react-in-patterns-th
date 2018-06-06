# คอมโพเนนต์สำหรับแสดงผล และ คอมโพเนนต์สำหรับจัดการข้อมูล

ในการเริ่มต้นทุกๆอย่าง มักจะยากเสมอ. React ไม่มีการดักจับข้อผิดพลาด และในฐานะผู้เริ่มต้นพวกเราทุกคนต่างก็มีคำถามมากหมาย สิ่งที่ผมคิดก็คือผมจะเอาข้อมูลไปไว้ตรงไหน ซึ่งข้อมูลมันสือสารหรือเปลี่ยนแปลงยังไงละ แล้วะจัดการกับสถานะของข้อมูลยังไง  คำถามเหล่านี้เป็นเรื่องสำคัญมากๆของบริบทหน้าที่ และในบางครั้งต้องอาศัยประสบการณ์และการฝึกในกับ React. อย่างไรก็ตาม, ยังมีรูปแบบถูกนำมาใช้อย่างแพร่หลาย และมันยังช่วงจัดระเบียบพื้นฐานแอพพลิเคชั่นจาก React  -  ไปจนถึงการแยกคอมโพเนนต์ ทั้ง คอมโพเนนต์สำหรับแสดงผล และ คอมโพเนนต์สำหรับจัดการข้อมูล.

เรามาเริ่มต้นกับตัวอย่างง่ายๆซึ่งจะแสดงให้เห็นถึงปัญหาในการแยกคอมโพเนนต์ โดยพวกเราจะใช้ คอมโพเนนต์ชื่อว่า 'Clock' ซึ่งได้รับการยอมรับ  [วันที่](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date) ซึ่งเป็นวัตุที่เป็นทั้ง prop และตัวแสดงผล ของเวลาแบบเรียลไทม์

```js
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = { time: this.props.time };
    this._update = this._updateTime.bind(this);
  }
  render() {
    const time = this._formatTime(this.state.time);
    return (
      <h1>
        { time.hours } : { time.minutes } : { time.seconds }
      </h1>
    );
  }
  componentDidMount() {
    this._interval = setInterval(this._update, 1000);
  }
  componentWillUnmount() {
    clearInterval(this._interval);
  }
  _formatTime(time) {
    var [ hours, minutes, seconds ] = [
      time.getHours(),
      time.getMinutes(),
      time.getSeconds()
    ].map(num => num < 10 ? '0' + num : num);

    return { hours, minutes, seconds };
  }
  _updateTime() {
    this.setState({
      time: new Date(this.state.time.getTime() + 1000)
    });
  }
};

ReactDOM.render(<Clock time={ new Date() }/>, ...);
```
ใน constructor ของคอมโพเนนต์นั้นพวกเราได้เริ่มต้น คอมโพเนนต์ state ซึ่งในกรณีนี้ เป็นเพียงแค่ค่า'time'(เวลา)ในปัจจุบัน โดยใช้ 'setInterval' พวกเราได้อัพเดท state ทุกๆวินาที และคอมโพเนนนั้นมีการรีเรนเดอร์ด้วย เพื่อให้ดูเหมือนนาฬิกาจริง เพื่อให้ดูเหมือนนาฬิกาจริง พวกเราได้ใช้ ตัวช่วย 2 method คือ `_formatTime` และ `_UpdateTime`. 
อันดับแรกคือการแยกองค์ประกอบ ชั่วโมง นาที และวินาที และเพื่อที่จะแน่ใจกับมันต้องติดตามรูปแบบตัวเลข 2หลัก  
`_updateTime` คือการทำให้ วัตถุเวลา`time`เป็นปัจุบัน ในทุกๆ 1วินาที
 
## ปัญหาที่เกิดขึ้น
ทั้งสองคอมโพเนนต์ของเรา ดูเหมือนจะมีภาระหน้าที่ที่มากเกินไป

* มันอัพเดท state ของมันด้วยตัวมันเอง การเปลี่ยนแปลงเวลาภายในคอมโพเนนตค์นั้น อาจจะไม่ใช่ความคิดที่ดีเพราะว่า เมื่อ `Clock` มันจะรู้แค่ค่าปัจุบันของมันเท่านั้น อ้าวแล้วถ้ามีส่วนอื่นของระบบซึ่งมันขึ้นอยู่กับข้อมูลที่ซึ่งมันยากที่จะแชร์กันละ 

* `_formatTime` is actually doing two things - it extracts the needed information from the date object and makes sure that the values are always presented by two digits. That's fine but it will be nice if the extracting is not part of the function because then it is bound to the type of the `time` object (coming as a prop). I.e. knows specifics about the shape of the data and at the same time deals with the visualization of it.

## Extracting the container

Containers know about data, its shape and where it comes from. They know details about how the things work or the so called *business logic*. They receive information and format it so like is easy to be used by the presentational component. Very often we use [higher-order components](https://github.com/krasimir/react-in-patterns/tree/master/patterns/higher-order-components) to create containers because they provide a buffer space where we can insert custom logic.

Here's how our `ClockContainer` looks like:

<span class="new-page"></span>

```js
// Clock/index.js
import Clock from './Clock.jsx'; // <-- that's the presentational component

export default class ClockContainer extends React.Component {
  constructor(props) {
    super(props);
    this.state = { time: props.time };
    this._update = this._updateTime.bind(this);
  }
  render() {
    return <Clock { ...this._extract(this.state.time) }/>;
  }
  componentDidMount() {
    this._interval = setInterval(this._update, 1000);
  }
  componentWillUnmount() {
    clearInterval(this._interval);
  }
  _extract(time) {
    return {
      hours: time.getHours(),
      minutes: time.getMinutes(),
      seconds: time.getSeconds()
    };
  }
  _updateTime() {
    this.setState({
      time: new Date(this.state.time.getTime() + 1000)
    });
  }
};
```

It still accepts `time` (a date object), does the `setInterval` loop and knows details about the data (`getHours`, `getMinutes` and `getSeconds`). In the end renders the presentational component and passes three numbers for hours, minutes and seconds. There is nothing about how the things look like. Only *business logic*.

## Presentational component

Presentational components are concerned with how the things look. They have the additional markup needed for making the page pretty. Such components are not bound to anything and have no dependencies. Very often implemented as a [stateless functional components](https://facebook.github.io/react/blog/2015/10/07/react-v0.14.html#stateless-functional-components) they don't have internal state.

In our case the presentational component contains only the two-digits check and returns the `<h1>` tag:

```js
// Clock/Clock.jsx
export default function Clock(props) {
  var [ hours, minutes, seconds ] = [
    props.hours,
    props.minutes,
    props.seconds
  ].map(num => num < 10 ? '0' + num : num);

  return <h1>{ hours } : { minutes } : { seconds }</h1>;
};
```

## Benefits

Splitting the components in containers and presentation increases the reusability of the components. Our `Clock` function/component may exist in application that doesn't change the time or it's not working with JavaScript's [Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date) object. That's because it is pretty *dummy*. No details about the data are required.

The containers encapsulate logic and we may use them together with different renderers because they don't leak information about the visual part. The approach that we took above is a good example of how the container doesn't care about how the things look like. We may easily switch from digital to analog clock and the only one change will be to replace the `<Clock>` component in the `render` method.

Even the testing becomes easier because the components have less responsibilities. Containers are not concern with UI. Presentational components are pure renderers and it is enough to run expectations on the resulted markup.

## Final thoughts

The concept of container and presentation is not new at all but it fits really nicely with React. It makes our applications better structured, easy to manage and scale.
