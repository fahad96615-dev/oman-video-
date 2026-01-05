  <!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<title>شرطة عُمان السلطانية – اتصال فيديو للصم والبكم</title>

<style>
body {
    margin: 0;
    font-family: "Segoe UI", Tahoma, Arial;
    background: linear-gradient(135deg, #f2f6fa, #e6ecf3);
    direction: rtl;
    text-align: center;
}

/* رأس الصفحة */
.header {
    background: #002d62;
    color: white;
    padding: 25px 15px;
}

.header h1 {
    margin: 0;
    font-size: 26px;
}

.header p {
    margin-top: 8px;
    font-size: 16px;
    opacity: 0.9;
}

/* الحاوية الرئيسية */
.container {
    max-width: 1000px;
    margin: 30px auto;
    padding: 20px;
}

/* الفيديو */
.videos {
    display: flex;
    justify-content: center;
    flex-wrap: wrap;
    gap: 20px;
}

video {
    width: 420px;
    max-width: 100%;
    height: 300px;
    background: black;
    border-radius: 15px;
    box-shadow: 0 6px 20px rgba(0,0,0,0.15);
}

/* الأزرار */
.controls {
    margin-top: 30px;
}

button {
    padding: 14px 26px;
    font-size: 17px;
    border: none;
    border-radius: 10px;
    cursor: pointer;
    margin: 10px;
    min-width: 180px;
}

.start {
    background: #198754;
    color: white;
}

.end {
    background: #dc3545;
    color: white;
}

/* تذييل */
.footer {
    margin-top: 40px;
    padding: 15px;
    font-size: 14px;
    color: #555;
}
</style>
</head>
<body>

<!-- العنوان -->
<div class="header">
    <h1>شرطة عُمان السلطانية</h1>
    <p>خدمة الاتصال المرئي للصم والبكم</p>
</div>

<!-- المحتوى -->
<div class="container">

    <div class="videos">
        <video id="localVideo" autoplay muted></video>
        <video id="remoteVideo" autoplay></video>
    </div>

    <div class="controls">
        <button class="start" onclick="startCall()">بدء الاتصال</button>
        <button class="end" onclick="endCall()">إنهاء الاتصال</button>
    </div>

</div>

<div class="footer">
    هذه الخدمة مخصصة للتواصل المرئي في الحالات الطارئة – لا يتم حفظ أي بيانات
</div>

<script>
let pc;
let localStream;

const localVideo = document.getElementById("localVideo");
const remoteVideo = document.getElementById("remoteVideo");

async function startCall() {
    pc = new RTCPeerConnection({
        iceServers: [{ urls: "stun:stun.l.google.com:19302" }]
    });

    pc.ontrack = e => remoteVideo.srcObject = e.streams[0];

    localStream = await navigator.mediaDevices.getUserMedia({
        video: true,
        audio: true
    });

    localVideo.srcObject = localStream;
    localStream.getTracks().forEach(track => pc.addTrack(track, localStream));

    pc.onicecandidate = e => {
        if (!e.candidate) {
            location.hash = btoa(JSON.stringify(pc.localDescription));
            alert("يرجى نسخ الرابط وإرساله للطرف الآخر");
        }
    };

    if (location.hash) {
        const offer = JSON.parse(atob(location.hash.substring(1)));
        await pc.setRemoteDescription(offer);
        const answer = await pc.createAnswer();
        await pc.setLocalDescription(answer);
    } else {
        const offer = await pc.createOffer();
        await pc.setLocalDescription(offer);
    }
}

function endCall() {
    if (pc) pc.close();
    if (localStream) localStream.getTracks().forEach(t => t.stop());
    localVideo.srcObject = null;
    remoteVideo.srcObject = null;
    location.hash = "";
}
</script>

</body>
</html>
