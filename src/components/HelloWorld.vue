<template>
  <div id="app" class="container">
    <div class="jumbotron">
      <h1>Hello from ROSDS!</h1>
    </div>

    <div class="row" style="max-height: 200px">
      <div class="col-md-6">
        <h3>Connection status</h3>

        <p class="text-success" v-if="connected">Connected!</p>
        <p class="text-danger" v-else>Not connected!</p>

        <label>Websocket server address</label>
        <input type="text" v-model="ws_address" />
        <br />
        <button @click="disconnect" class="btn btn-danger" v-if="connected">Disconnect!</button>
        <button @click="connect" class="btn btn-success" v-else>Connect!</button>
        <button @click="startStream" class="btn btn-success">Start!</button>
      </div>
      <div class="col-md-6" style="max-height:200px; overflow:auto;">
        <h3>Log messages</h3>
        <div>
          <p v-for="log in logs">
            {{ log }}
          </p>
        </div>
      </div>
    </div>
    <hr>

    <div class="row">
      <div class="col-md-6 row">
        <div class="col-md-12 text-center">
          <h5>Commands</h5>
        </div>

        <!-- 1st row -->
        <div class="col-md-12 text-center">
          <button @click="forward" :disabled="loading || !connected" class="btn btn-primary">Go straight</button>
          <br><br>
        </div>

        <!-- 2nd row -->
        <div class="col-md-4 text-center">
          <button @click="turnLeft" :disabled="loading || !connected" class="btn btn-primary">Turn left</button>
        </div>
        <div class="col-md-4 text-center">
          <br>
          <button @click="stop" :disabled="loading || !connected" class="btn btn-danger">Stop</button>
          <br>
          <br>
        </div>
        <div class="col-md-4 text-center">
          <button @click="turnRight" :disabled="loading || !connected" class="btn btn-primary">Turn right</button>
        </div>

        <!-- 3rd row -->
        <div class="col-md-12 text-center">
          <button @click="backward" :disabled="loading || !connected" class="btn btn-primary">Go straight</button>
        </div>
      </div>

      <div class="col-md-6">
        camera
        <!-- <div id="mjpeg"></div> -->
        <!-- <img id="imgView" :src="imageSrc" height="600" width="800" /> -->
        <img id="imgView"  height="600" width="800" />
      </div>
    </div>
  </div>
</template>

<script>
import ROSLIB from 'roslib';
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  // storing the state of the page
  data() {
    return {
      connected: false,
      ros: null,
      ws_address: 'ws://127.0.0.1:9090',
      logs: [],
      error: null,
      cameraTopic: null,
      imageSrc: ""
    }
  },
  mounted() {



  },
  methods: {
    // setCamera() {
    //   console.log('set camera method')
    //   this.cameraViewer = new MJPEGCANVAS.Viewer({
    //     divID: 'mjpeg',
    //     host: '127.0.0.1',
    //     width: 640,
    //     height: 480,
    //     topic: '/my_camera/image_raw',
    //     port: 9090,
    //   })
    // },
    connect: function () {
      this.logs.unshift('connect to rosbridge server!!')
      this.ros = new ROSLIB.Ros({
        url: this.ws_address
      })
      this.ros.on('connection', () => {
        // this.setCamera();
        this.connected = true
        this.logs.unshift('Connected!')

        var videoStream = new ROSLIB.Topic({
          ros: this.ros,
          name: '/image_raw/compressed',
          messageType: 'sensor_msgs/msg/CompressedImage'
        });

        videoStream.subscribe(function (message) {
          // var data = `data:image/jpg;base64, ` + message.data;
          // document.getElementById('imgView').src = data;

          // this.imageSrc = `data:image/jpg;base64, ` + message.data;
          // console.log('videoStream: ', this.imageSrc)
        document.getElementById('imgView').src = `data:image/jpg;base64, ` + message.data;

    //   console.log('DONE VIDEO!')


        })


      })
      this.ros.on('error', (error) => {
        this.logs.unshift('Error connecting to websocket server')
        // console.log('Error connecting to websocket server: ', error)
      })
      this.ros.on('close', () => {
        this.connected = false
        this.logs.unshift('Connection to websocker server closed')
        // console.log('Connection to websocket server closed.')
      })
    },
    disconnect: function () {
      this.ros.close()
    },
    // startStream() {
    //   // alert('a');

    //   //       document.getElementById('imgView').src = `data:image/png;base64, iVBORw0KGgoAAAANSUhEUgAAAAUA
    //   // AAAFCAYAAACNbyblAAAAHElEQVQI12P4//8/w38GIAXDIBKE0DHxgljNBAAO
    //   //     9TXL0Y4OHwAAAABJRU5ErkJggg==`;

    //   var videoStream = new ROSLIB.Topic({
    //     ros: this.ros,
    //     name: '/my_camera/compressed_image',
    //     messageType: 'sensor_msgs/msg/Image'
    //   });

    //   videoStream.subscribe(function (message) {
        
    //     document.getElementById('imgView').src = `data:image/jpg;base64, ` + message.data;


    //     //         this.imageSrc = `data:image/png;base64, iVBORw0KGgoAAAANSUhEUgAAAAUA
    //     // AAAFCAYAAACNbyblAAAAHElEQVQI12P4//8/w38GIAXDIBKE0DHxgljNBAAO
    //     //     9TXL0Y4OHwAAAABJRU5ErkJggg==`;
    //   })
    //   console.log('DONE VIDEO!')
    // },
    etTopic: function () {
      this.topic = new ROSLIB.Topic({
        ros: this.ros,
        name: '/cmd_vel',
        messageType: 'geometry_msgs/Twist'
      })
    },
    forward: function () {
      this.message = new ROSLIB.Message({
        linear: { x: 1, y: 0, z: 0, },
        angular: { x: 0, y: 0, z: 0, },
      })
      this.setTopic()
      this.topic.publish(this.message)
    },
    stop: function () {
      this.message = new ROSLIB.Message({
        linear: { x: 0, y: 0, z: 0, },
        angular: { x: 0, y: 0, z: 0, },
      })
      this.setTopic()
      this.topic.publish(this.message)
    },
    backward: function () {
      this.message = new ROSLIB.Message({
        linear: { x: -1, y: 0, z: 0, },
        angular: { x: 0, y: 0, z: 0, },
      })
      this.setTopic()
      this.topic.publish(this.message)
    },
    turnLeft: function () {
      this.message = new ROSLIB.Message({
        linear: { x: 0.5, y: 0, z: 0, },
        angular: { x: 0, y: 0, z: 0.5, },
      })
      this.setTopic()
      this.topic.publish(this.message)
    },
    turnRight: function () {
      this.message = new ROSLIB.Message({
        linear: { x: 0.5, y: 0, z: 0, },
        angular: { x: 0, y: 0, z: -0.5, },
      })
      this.setTopic()
      this.topic.publish(this.message)
    },
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
