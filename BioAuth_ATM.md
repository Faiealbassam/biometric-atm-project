
# BioAuth ATM System

The **BioAuth ATM System** is a secure, cardless ATM solution that uses **facial recognition**, **fingerprint scanning**, and a **custom PIN** to authenticate users.



## Collaborators
This project was developed by:
- **Faie Albassam**
- **Dalal Almubarak**
- **Noura Alzuabi**
- **Hayat Batook**

As part of the senior graduation project at **Prince Mohammad Bin Fahd University** (PMU).



##  Key Features
- Replaces ATM cards and PINs with facial and fingerprint authentication
- Uses a web interface for camera and fingerprint reader access
- Real-time biometric capture and validation using JavaScript and AJAX
- Includes a secure PIN setup option



## Technologies Used
- HTML5 / JavaScript / jQuery
- AJAX for client-server communication
- Canvas API for image capture
- Fingerprint controller for biometric scanning



## Implementation Code

### Face ID Registration
```html
<script>
var mediaStream = null;
var constraints = {
    audio: false,
    video: {
        width: { ideal: 640 },
        height: { ideal: 480 },
        facingMode: "environment"
    }
};

async function getMediaStream(constraints) {
    try {
        mediaStream = await navigator.mediaDevices.getUserMedia(constraints);
        let video = document.getElementById('cam');
        video.srcObject = mediaStream;
        video.onloadedmetadata = (event) => { video.play(); };
    } catch (err) {
        console.error(err.message);
    }
};

async function switchCamera(cameraMode) {
    if (mediaStream && mediaStream.active) {
        mediaStream.getVideoTracks().forEach(track => track.stop());
    }
    document.getElementById('cam').srcObject = null;
    constraints.video.facingMode = cameraMode;
    await getMediaStream(constraints);
}

function takePicture() {
    let canvas = document.getElementById('canvas');
    let video = document.getElementById('cam');
    let context = canvas.getContext('2d');
    const width = video.videoWidth;
    const height = video.videoHeight;
    if (width && height) {
        canvas.width = width;
        canvas.height = height;
        context.drawImage(video, 0, 0, width, height);
        var data = canvas.toDataURL('image/png');
        document.getElementById('photo').src = data;
        document.getElementById("loadingOverlay").style.display = "flex";
        $.ajax({
            url: "{{ url_for('auth.register_face') }}",
            type: 'POST',
            data: JSON.stringify({ email: registeredEmail, imgData: data }),
            contentType: "application/json",
            success: function (response) {
                document.querySelectorAll(".step").forEach(step => step.classList.remove("active"));
                document.getElementById("SuccessfulRegister").classList.add("active");
                document.getElementById("loadingOverlay").style.display = "none";
            },
            error: function () {
                document.getElementById("loadingOverlay").style.display = "none";
            }
        });
    }
}

function clearPhoto() {
    let canvas = document.getElementById('canvas');
    let context = canvas.getContext('2d');
    context.fillStyle = "#AAA";
    context.fillRect(0, 0, canvas.width, canvas.height);
    document.getElementById('photo').src = canvas.toDataURL('image/png');
}

document.getElementById('loginFaceBtn').onclick = () => switchCamera("user");
document.getElementById('snapBtn').onclick = (e) => { takePicture(); e.preventDefault(); };
clearPhoto();
</script>




### Fingerprint Registration
```html
<script>
var registeredEmail = null;
let resultImg = document.getElementById('resultImg');

function fromBase64Url(s) {
    return ((s.length % 4 === 2) ? s + "==" : (s.length % 4 === 3) ? s + "=" : s)
        .replace(/-/g, "+").replace(/_/g, "/");
}

window.addEventListener('DOMContentLoaded', function () {
    let reader = new fpController({ debug: true, version: 1 });
    document.getElementById('startReading').addEventListener('click', () => reader.startReading());

    reader.reader.on("SamplesAcquired", (event) => {
        let samples = event.samples[0];
        sendSamplesToServer(samples);
    });
});

function sendSamplesToServer(samples) {
    var base64Str = "data:image/png;base64," + fromBase64Url(samples);
    resultImg.src = base64Str;
    document.getElementById("loadingOverlay").style.display = "flex";

    $.ajax({
        type: "POST",
        url: "{{ url_for('auth.register_finger') }}",
        data: JSON.stringify({ email: registeredEmail, img: base64Str }),
        contentType: "application/json",
        success: function () {
            document.querySelectorAll(".step").forEach(step => step.classList.remove("active"));
            document.getElementById("SuccessfulRegister").classList.add("active");
            document.getElementById("loadingOverlay").style.display = "none";
        },
        error: function () {
            alert("Fingerprint registration failed");
            document.getElementById("loadingOverlay").style.display = "none";
        }
    });
}
</script>



### PIN Registration
```html
<script>
$(document).ready(function () {
    $('.tick-btn').click(async function () {
        var pin = $('#pinInput').val();
        var userId = 1;
        document.getElementById("loadingOverlay").style.display = "flex";

        try {
            await new Promise((resolve, reject) => {
                $.ajax({
                    type: "POST",
                    url: "{{ url_for('auth.register_pin') }}",
                    contentType: "application/json",
                    data: JSON.stringify({ pin: pin, user_id: userId, email: registeredEmail }),
                    success: function () { resolve(); },
                    error: function (xhr) {
                        alert(JSON.parse(xhr.responseText).error);
                        window.location.reload();
                        reject(new Error("Registration failed"));
                    }
                });
            });
            document.getElementById("loadingOverlay").style.display = "none";
        } catch (error) {
            console.error("An error occurred:", error.message);
        }
    });
});
</script>
