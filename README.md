## Integrating with Listen.Doctor's Real-Time API Using Socket.IO

This document provides a comprehensive guide to integrating Listen.Doctor's real-time API through socket connections, compatible with any language supporting Socket.IO. The aim is to establish a real-time communication channel with the Listen.Doctor platform, allowing developers to send and receive audio data in chunks and responses from background tasks. Below, we will explore the process of setting up and using Listen.Doctor's sockets API.

### Overview
Listen.Doctor's real-time integration leverages Socket.IO to facilitate real-time audio streaming and message exchange. By using this approach, a client can connect to the Listen.Doctor server, authenticate using a token, and then start recording and transmitting audio chunks for processing. The server provides responses related to processing status and other operational messages. Socket.IO's cross-platform nature means this integration approach works with numerous languages and frameworks, such as JavaScript, Python, Java, Swift, and others.

### A fully functional project with comprehensive comments.

[Click Here and look at source!](https://api-beta.listen.doctor/sample-sockets)

### Prerequisites
- **Socket.IO library**: Ensure you have installed the appropriate Socket.IO client library for your target language. Example libraries include JavaScript's Socket.IO, Python-SocketIO, and similar ones for Java, Swift, etc.
- **Authentication**: To interact with Listen.Doctor, you'll need an API key, client ID, and client secret. These credentials are used to fetch an authentication token from the Listen.Doctor API, which is required to establish socket communication.

### Connection Flow
The process of connecting to Listen.Doctor using Socket.IO involves the following steps:

1. **Fetch Authentication Token**: Before establishing a socket connection, authenticate yourself with the Listen.Doctor API using your credentials. This is done via an HTTP POST request to `/v1/iam` to obtain an access token.

    Example request:

    ```http
    POST https://api-beta.listen.doctor/v1/iam
    Headers:
      Content-Type: application/json
      x-api-key: <API_KEY>
    Body:
    {
      "client_id": "<CLIENT_ID>",
      "client_secret": "<CLIENT_SECRET>",
      "grant_type": "client_credentials",
      "doctor": "<DOCTOR_UUID>"
    }
    ```

    The response will include a `token` that must be used in subsequent socket connections.

2. **Establish Socket Connection**: Using the obtained token, establish a connection to Listen.Doctor's Socket.IO endpoint (`http://api-beta.listen.doctor`). This is done by providing the token in the `authorization` field of the connection options.

    ```javascript
    socket = io("http://api-beta.listen.doctor", {
      auth: {
        authorization: 'Bearer ' + token
      }
    });
    ```

    Upon successful connection, the server will emit a `connect` event, which should be handled appropriately by the client.

3. **Join Room**: To begin streaming data, you need to join a room. Emit a `join_room` event specifying a unique room identifier.

    ```javascript
    socket.emit('join_room', { room: '<ROOM_UUID>' });
    ```

    After successfully joining the room, you will receive a `room_joined` event, allowing you to proceed with recording.

### Real-Time Audio Streaming
Listen.Doctor's API allows real-time audio streaming by periodically sending audio data to the server. Here is how to handle audio data:

1. **Start Recording**: Use the MediaRecorder API (in JavaScript) or the equivalent library in other languages to capture audio. Define an interval (e.g., every 10 seconds) to capture chunks of audio and transmit them to the server.

    ```javascript
    mediaRecorder.start();
    chunkInterval = setInterval(() => {
      mediaRecorder.requestData();
      sendAudioChunk();
    }, trunkSize);
    ```

2. **Send Audio Chunks**: Once audio data is available, it should be packaged into a `Blob` and sent to the server via the `audio_chunk` event.

    ```javascript
    const audioBlob = new Blob(audioChunks, { type: 'audio/wav' });
    const reader = new FileReader();
    reader.readAsArrayBuffer(audioBlob);
    reader.onload = () => {
      const buffer = new Uint8Array(reader.result);
      socket.emit('audio_chunk', buffer, (response) => {
        if (response.status === 'success') {
          console.log('Audio chunk sent successfully');
        } else {
          console.error('Error sending audio chunk:', response.error);
        }
      });
    };
    ``
    Clear the `audioChunks` array after sending each chunk.

3. **Stop Recording**: Stop the recording process and notify the server by emitting the `stop_recording` event.

    ```javascript
    mediaRecorder.stop();
    socket.emit('stop_recording');
    ```

### Handling Server Responses
The Listen.Doctor server provides various response events:

- **`recording_started`**: Indicates that the server has successfully started recording.
- **`processing_complete`**: Informs the client that audio processing is complete and results are available.
- **`error`**: If an error occurs during communication or processing, the server emits an error event. Always handle these gracefully to provide a good user experience.

### Best Practices
- **Error Handling**: Ensure to listen to the `error` event on the socket to gracefully handle any issues.
- **Token Management**: Tokens are time-bound; ensure to refresh the token when it expires.
- **Chunk Size Configuration**: Adjust the chunk size (`trunkSize`) based on the MIME type used. This ensures compatibility and better streaming performance.

### Supported Languages and Libraries
Listen.Doctor's socket server supports integration with various languages through Socket.IO. Below are some common clients:
- **JavaScript**: [Socket.IO](https://socket.io/)
- **Python**: [Python-SocketIO](https://python-socketio.readthedocs.io/en/latest/)
- **PHP**: [PHP Socket.IO](https://dhtml.github.io/phpsocket.io/)
- **Swift**: [Swift Socket.IO](https://github.com/socketio/socket.io-client-swift)
- **Java (Android)**: [Java Socket.IO](https://github.com/socketio/socket.io-client-java)

These libraries allow you to integrate Listen.Doctor's real-time capabilities into a variety of applications and platforms, providing a robust and versatile solution for real-time medical data communication.

### A fully functional project with comprehensive comments.

[Click Here and look at source!](https://api-beta.listen.doctor/sample-sockets)

### Conclusion
Integrating Listen.Doctor's real-time API using Socket.IO allows for seamless audio streaming and interaction with the platform. With support for multiple languages and frameworks, developers can leverage this API to build interactive and responsive healthcare applications that capture and analyze patient data in real time.

