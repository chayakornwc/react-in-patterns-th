# Presentational component และ Container component

ในการเริ่มต้นทุกๆอย่าง มักจะยากเสมอ. React เองนั้น ไม่มีการดักจับข้อผิดพลาดและในฐานะผู้เริ่มต้นพวกเราทุกคนต่างก็มีคำถามมากมาย สิ่งที่ผมคิดก็คือผมจะเอาข้อมูลไปไว้ตรงไหน แล้วข้อมูลมันสื่อสารหรือเปลี่ยนแปลงยังไงละ แล้วะจัดการกับ state ของข้อมูลยังไง คำถามเหล่านี้เป็นเรื่องสำคัญมากๆทั้งบริบทและหน้าที่แม้ในบางครั้งก็ต้องอาศัยประสบการณ์และการฝึกฝนกับเจ้า React. อย่างไรก็ตามยังมีรูปแบบที่ถูกนำมาใช้อย่างแพร่หลายและมันยังช่วยจัดระเบียบพื้นฐานแอพพลิเคชั่นจากที่พัฒนา React  -  ไปจนถึงการแยก Component ต่างๆ ไม่ว่าจะเป็น Presentational component และ Container component.

เรามาเริ่มต้นกับตัวอย่างง่ายๆซึ่งจะแสดงให้เห็นถึงปัญหาในการ Component โดยเราจะใช้ Component `Clock` ซึ่งรับ oject [วันที่](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date) ในฐานะ  prop และ display ของเวลาแบบเรียลไทม์

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
ใน constructor ของคอมโพเนนต์นั้นเราได้เริ่มต้น state ของ Components  ซึ่งในกรณีนี้ เป็นเพียงแค่ค่า `เวลา` ในปัจจุบัน โดยใช้ `setInteval` ซึ่งมีการอัพเดท state ทุกๆวินาที และตัว Component เองนั้นก็ยังมีการ render ใหม่ด้วยเพื่อให้ดูเหมือนนาฬิกาจริง เราได้ใช้ตัวช่วย 2 method คือ `_formatTime` และ `_UpdateTime`. 
อันดับแรกคือการแยกส่วนย่อย ชั่วโมง นาที และวินาที และเพื่อที่จะแน่ใจกับมันต้องติดตามรูปแบบตัวเลข 2หลัก  
`_updateTime` คือการทำให้ Object `time`เป็นปัจุบัน ในทุกๆ 1วินาที
 
## ปัญหาที่เกิดขึ้น

ทั้งสอง Component ของเรา ดูเหมือนจะมีภาระหน้าที่ที่มากเกินไป
* มันอัพเดท state ด้วยตัวมันเอง การเปลี่ยนแปลงเวลาภายใน Component นั้น อาจจะไม่ใช่ความคิดที่ดีเพราะ `Clock` มันจะรู้แค่ค่าปัจุบันของมันเท่านั้น อ้าวแล้วถ้ามีส่วนอื่นของระบบซึ่งมันขึ้นอยู่กับข้อมูลที่ซึ่งมันยากที่จะแชร์กันละ 
* `_formatTime` เป็น สองอย่างที่ต้องการ การแตกย่อยของข้อมูล จาก data object และเพื่อให้แน่ใจว่ามันจะมีค่าตัวเลขสองหลักที่จะแสดงผลออกมาเสมอ
อย่างไรก็ตาม เรายังสามารถที่จะแตกย่อยส่วนที่เป็นฟังก์ชั่นได้ดี เพราะเมื่อมันสัมพันธ์กับชนิดของ  object `time`  จาก object `date` มาเป็น property ซึ่งเป็นที่รู้จักในฐานะของความเฉพาะของข้อมูล และในบางเวลา มันก็เป็นที่รู้จักในนามของการจำลองภาพหรือจินตภาพ


## การแยกส่วนย่อยของ Container

Containers หรือเป็นที่รู้จักในนามของข้อมูลซึ่งมีรูปร่างและที่มาซึ่งหลายคนคงทราบถึงรายละเอียดและการทำงานของมัน หรือจะเรียกอีกอย่างว่า bussiness logic ซึ่งมันได้รับ รูปแบบและข้อมูลที่ดูเหมือนง่ายโดยการใช้ Presentational component, เราได้ใช้ [higher-order components]  (https://github.com/krasimir/react-in-patterns/tree/master/patterns/higher-order-components) ในการสร้าง ontainer บ่อยมากเพราะมันให้พื้นที่ buffer ซึ่งเราจะสามารถเพิ่มการจัดการข้อมูลด้วยตัวเองได้  


นี่คือ `ClockContainer` ลองดูที่:

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
มันยังคงรับ `เวลา` (date object)   รายละเอียดของข้อมูลที่เป็นเวลา (`getHours`, `getMinutes` และ `getSeconds`) จาก ลูป `setInterval`  ในตอนการจบ render ของ Component presentational นั้น จะส่งผ่านตัวเลข 3 ตัวก็คือ ชั่วโมง นาที และวินาที มันดูเหมือนจะไม่มีมีอะไรเกิดขึ้นในการมองหาเพียงแค่ `bussiness logic`เท่านั้น



## Presentational component
Presentational Component เป็นสิ่งที่ดูน่าสนใจ มันยังต้องการ ปรับแต่งเพิ่มเติมในการทำให้ หน้าต่างๆดูสวยงาม ยกตัวอย่าง Component ต่างๆที่ไม่ได้มีความสัมพันธ์กับสิ่งใดๆก็ตามและยังไม่ได้เกี่ยวข้องกับ dependencies ใดๆ บ่อยครั้งมากที่ต้องมีการดำเนินการกับมัน ในฐานนะ [stateless functional components](https://facebook.github.io/react/blog/2015/10/07/react-v0.14.html#stateless-functional-components) ซึ่งมันไม่ได้มี state ภายในของมันเอง

ในกรณีของเรา มีแค่ Presentational component เท่านั้น ซึ่งมีการตรวจสอบตัวเลขสองหลัก ละ return ออกมา ใน tag `<h1>` 

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


##  ประโยชน์ที่ได้รับ
การแยก  component ต่างๆนั้น ทั้งใน container component และ presentation component และนำ Component มาใช้อีกครั้งบ่อยๆนั้น 
  ยกตัวอย่างของเรา `Clock` คือฟังก์ชั่นหรือ Component ในเวลาเดียวกัน อาจจะมีมีอยู่ใน application ซึ่งมันไม่เปลี่ยนแปลงเวลา หรือไม่ทำงานด้วย Oject [Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date) ใน Javascirpt เพราะว่ามันคือ *dummy*ที่สวยงาม และไม่มีรายละเอียดเกี่ยวกับข้อมูลที่จำเป็น
  
Container ต่างๆ ที่ encapsulate logic และเราอาจจะใช้มันร่วมกันซึ่งมันยากต่อการ render ออกมา เพราะข้อมูลมันไม่มีจุดรั่วเกี่ยวกับส่วนจำลอง, การที่นำมายกตัวอย่างที่ดี ของการใช้ container โดยไม่สนใจว่ามันจะมีลักษณะต่างๆยังไง เราหวังว่ามันจะเปลี่ยนจาก นาฬิกาดิติตอลไปเป็น นาฬิกาอนาล็อก อย่างง่ายดายเพียงเท่านั้นมันยังจะถูกที่แทนด้วย Component `Clock` ใน method `render`

แม้แต่การทดสอบยังสามารถที่ทำได้ง่ายขึ้น Component ต่างๆยังมีหน้าที่ที่น้อยลง หรือไม่มีเลย อีกทั้งยังไม่จำเป็นต้องกังวลกับเรื่อง UI
Presentational component ต่างๆนั้นมีการ render สิ่งต่างๆออกมาได้อย่างดิบๆ และ ยังเราเดาถึงผลของการออกแบบหน้าตาได้

## ความคิดสุดท้าย
แนวคิดสุดท้าย ของ container และ presentation นั้นไม่ใช่เรื่องใหม่ แต่ทุกอย่างเป็นเรื่องที่เหมาะกับ React จริงๆซึ่งมันสามารถสร้างโครงสร้าง
Application ของเราให้ดีขึ้นีกทั้งยังมาสามารถจัดการและปรับปรุงขอบเขตได้ง่าย 
