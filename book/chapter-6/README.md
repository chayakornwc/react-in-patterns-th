# Presentational component และ Container component

ในการเริ่มต้นทุกๆอย่าง มักจะยากเสมอ. React เองนั้น ไม่มีการดักจับข้อผิดพลาด และในฐานะผู้เริ่มต้นพวกเราทุกคนต่างก็มีคำถามมากหมาย สิ่งที่ผมคิดก็คือผมจะเอาข้อมูลไปไว้ตรงไหน ซึ่งข้อมูลมันสือสารหรือเปลี่ยนแปลงยังไงละ แล้วะจัดการกับสถานะของข้อมูลยังไง  คำถามเหล่านี้เป็นเรื่องสำคัญมากๆของบริบทหน้าที่ และในบางครั้งต้องอาศัยประสบการณ์และการฝึกในกับ React. อย่างไรก็ตาม, ยังมีรูปแบบถูกนำมาใช้อย่างแพร่หลาย และมันยังช่วงจัดระเบียบพื้นฐานแอพพลิเคชั่นจาก React  -  ไปจนถึงการแยกคอมโพเนนต์ ทั้ง คอมโพเนนต์สำหรับแสดงผล และ คอมโพเนนต์สำหรับจัดการข้อมูล.

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
* `_formatTime` เป็น สองอย่างที่ต้องการ การแตกย่อยของข้อมูล จาก data object และเพื่อให้แน่ใจว่ามันจะมีค่าตัวเลขสองหลักที่จะแสดงผลออกมาเสมอ
อย่างไรก็ตาม เรายังสามารถที่จะแตกย่อยส่วนที่เป็นฟังก์ชั่นได้ดี เพราะเมื่อมันสัมพันธ์กับชนิดของ  object `time`  จาก object `date` มาเป็น property ซึ่งเป็นที่รู้จักในนามของ ความเฉพาะของข้อมูล และในบางเวลา มันก็เป็นที่รู้จักในนามของการจำลองภาพ


## การแยกส่วนย่อยของ container

Containers หรือเป็นที่รู้จักในนามข้อมูลซึ่งมีรูปร่างและที่มา ซึ่งหลายคนคงทราบถึงรายละเอียดและการทำงานของมัน หรือจะเรียกอีกอย่างว่า bussiness logic   ซึ่งมันได้รับ รูปแบบและข้อมูลที่ดูเหมือนง่ายโดยการใช้ presentational component, พวกเราได้ใช้ [higher-order components]  (https://github.com/krasimir/react-in-patterns/tree/master/patterns/higher-order-components) ในการสร้าง container บ่อยมากเพราะมันให้พื้นที่ buffer ซึ่งเราจะสามารถเพิ่มการจัดการข้อมูลด้วยตัวเองได้  


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
มันยังคงรับ `เวลา` (date object)   รายละเอียดของข้อมูลที่เป็นเวลา (`getHours`, `getMinutes` และ `getSeconds`) จาก ลูป setInterval  ในตอนการจบการrender presentational component  จะส่งผ่านตัวเลข 3 ตัวก็คือ ชั่วโมง นาที และวินาที มันดูเหมือนจะไม่มีมีอะไรเกิดขึ้นนะในการมองหาเพียงแค่ `bussiness logic` เท่านั้น



## Presentational component
Presentational Components เป็นสิ่งที่ดูน่าสนใจ มันยังต้องการmarkup เพิ่มเติม ในการทำให้ เพจต่างๆดูสวยงาม ยกตัวอย่าง component ต่างๆที่ไม่ได้มีความสัมพันธ์กับอะไรก็ตามและยังไม่ได้เกี่ยวข้องกับ dependencies ใดๆ บ่อบครั้งมากที่ต้องมีการดำเนินการกับมัน ในฐานนะ [stateless functional components](https://facebook.github.io/react/blog/2015/10/07/react-v0.14.html#stateless-functional-components) ซึ่งมันไม่ได้มี state ภายในของมันเอง

ในกรณีของเรา มีแค่ presentational component เท่านั้น ซึ่งมีการตรวจสอบตัวเลขสองหลัก ละ return ออกมา ใน tag `<h1>` 

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
การแยก  component ต่างๆนั้น ทั้งใน container component และ presentation component และการใช้นำคอมโพเนนต์ มา reuse บ่อยๆนั้น 
  ยกตัวอย่างของเรา `Clock` คือฟังก์ชั่นหรือคอมโพเนนต์ในเวลาเดียวกัน อาจจะมีมีอยู่ใน application ซึ่งมันไม่เปลี่ยนแปลงเวลา หรือไม่ทำงานด้วย Oject [Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date) ใน Javascirpt เพราะว่ามันคือ *dummy*ที่สวยงาม และไม่มีรายละเอียดเกี่ยวกับข้อมูลที่จำเป็น
  
Container ต่างๆ ที่ encapsulate logic และำวกเรา อาจจะใช้มันร่วมกันซึ่งมันยากต่อการ render ออกมา เพราะข้อมูลมันไม่มีจุดรั่วเกี่ยวกับส่วนจำลอง, การที่พวกพเราจะยอมรับมัน คือการที่นำมาเป็นตัวอย่างที่ดี ของการใช้ container โดยไม่สนใจว่ามันจะมีลักษณะต่างๆยังไง เราหวังว่ามันจะเปลี่ยนจาก นาฬิกาดิติตอลไปเป็น นาฬิกาอนาล็อก อย่างง่ายดายเพียงเท่านั้นมันยังจะถูกที่แทนด้วย Components `Clock` ใน method `render`

แม้แต่การทดสอบยังสามารถที่ทำได้ง่ายขึ้น components ต่างๆยังมีหน้าที่ที่น้อยลง หรือไม่มีเลย อีกทั้งยังไม่จำเป็นต้องกังวลกับเรื่อง UI
Presentational component ต่างๆนั้นมีการ renders สิ่งต่างๆออกมาได้อย่างดิบๆ และ ยังเราเดาถึงผลของการออกแบบหน้าตา

## ความคิดสุดท้าย
แนวคิดสุดท้าย ของ container และ presentation นั้น ไม่ใช่เรื่องใหม่ แต่ทุกอย่างเป็นเรื่อง แต่มันเหมาะกับ React จริงๆ ซึ่งมันสามารถสร้างโครงสร้าง
Application ของเราให้ดีขึ้นีกทั้งยังมาสามารถจัดการ และ ปรับปรุงขอบเขตได้ง่าย
