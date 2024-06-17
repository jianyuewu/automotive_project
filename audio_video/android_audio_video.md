# Audio  
AAC(Advanced Audio Coding).  
In most AAC encoder implementations, setting the bit rate to less than or equal to 48Kbps will automatically enable PS (Parametric Stereo) technology, while if it is greater than 48Kbps, PS will not be added, which is equivalent to ordinary HE-AAC (High Efficiency Advanced Audio Codec).  
Source is PCM data, calculation example:  
    If using 44100 sampling rate, dual channels, each sampling point is represented by 16 bits, then the size of 1 minute of CD quality PCM data is:  
    44100 * 2 * 16 * 60 / 8 = 10.09MB  
Android SDK (Java layer): MediaPlayer, SoundPool and AudioTrack.  
Android NDK (low latency): AAudio(>= Android 8.0) or OpenSL ES(< Android 8.0).  

AudioTrack： 
  1. Config: streamType，sampleRateInHz(44.1k for music, lower for voice)，channelConfig(mono in collect, play with stereo),  
    audioFormat(better 16bit), bufferSizeInBytes(set by getMinBufferSize func), mode(STATIC, STREAM).  
  2. Play: call play().  
  3. Start, will block.  
  4. Stop: call stop() and release().  

OpenSL ES:  
  1. Create object interface.  
  2. RealizeObject and use interface to handle the object.  
  3. GetInterface().  
  4. CreateOutputMix().  
  5. realizeObject with outputMixObject and audioPlayerObject.  
  6. SetPlayState().  
  7. destroyObject().  

Oboe:  
  1. AudioStreamBuilder(), AudioStreamDataCallback().  
  2. Play audio: requestStart().  
  3. close().  


# Video  
RGBA: Used in situations where image transparency needs to be handled, such as in image editing software, game development, and rendering engines.  
A stands for Alpha, which represents transparency.  
YUV: Used for video compression and transmission, such as in MPEG, JPEG, H.264, and others.  
Y: Luminance.  
U: Chrominance 1.  
V: Chrominance 2.  

H.266(VVC), H.265(HEVC), H.264(AVC): Ensuring video quality, minimize the video size, can save bandwidth. Defined by ITU and Mpeg.  
H.266 can support 16k video, while H.265/ H.264 can support 4k/ 8k.  
H.266 use new algorithm, has 30% to 50% higher compression rate than H.265.  
Camera -> H264Encoder -> Mux  
Recorder -> AACEncoder -> Mux  
Mux -> Mp4  

Android uses EGL (Embeded-System Graphics Library) to control OpenGL ES (Embed System).  

GLSurfaceView.  
```bash
ffmpeg -i input.mp4 -movflags faststart -acodec copy -vcodec copy output.mp4  
```
Better use Vulkan & Metal(>= Android 10.0), performance is better. If in legacy device, can still use OpenGL ES.  
OpenGL ES:  
  1. Specify the geometric object.  
  2. Vertex transform.  
  3. Element assembly.  
  4. Rasterization operations.  
  5. Fragment processing.  
  6. Framebuffer.  
glCreateShader(), glShaderSource(), glCompileShader(), glGetShaderiv(), glGetShaderInfoLog().  
glCreateProgram(), glAttachShader(), glLinkProgram(), glGetProgramiv(), glGetProgramInfoLog().  

RenderThread(OpenGL ES) -> EGLDisplay -> TextureView.  
Buffers: Back Frame Buffer, Front Frame Buffer.  
1. Display target: eglGetDisplay(), eglInitialize().  
2. Thread context: eglCreateContext(), eglGetConfigAttrib().  
3. Surface(Connect display): eglCreateWindowSurface(), eglCreatePbufferSurface().  
eglMakeCurrent(), eglSwapBuffers(), eglDestroySurface(), eglDestroyContext().  
eglSwapBuffer() can swap these two buffers.  

## GLSL  
glGenTextures(), glBindTexture(), glTexParameteri().  
glTexImage2D(), glReadPixels().  
glViewport(), glUseProgram().  
glDrawArrays(), glDeleteTextures().  

## Frame type  
I frame (Intra-Coded Frames): consists only macroblocks use intra-prediction.  
P frame (): predicted frame, allows macroblocks to be compressed using temporal prediction and spatial prediction.  
B frame: bi-directional frame, supports backward and forward prediction.  

inter-frame prediction temporal: It uses the block motion vector from different lists.  
```bash
ffplay -flags2 +export_mvs -vf codecview=mv=pf+bf+bb input.flv  
```
inter-frame prediction spatial: It predicts the movement from neighbour macroblocks in same frame.  

## H.264 software encoder with ffmpeg  
init(), createEncoder().  
Texture copy thread (setup open GL thread) -> VideoFrame Queue -> encode thread -> videoX264 encoder.  

## H.264 hardware encoder with mediacodec  
Android uses mediacode.  
CVPixelBuffer -> VTCompression Session (video Encoder) -> CMSampleBuffers.  
CVPixelBufferLockBaseAddress().  
CVPixelBufferRelease().  
IOS uses videoToolbox.  
If it is key frame (I frame), then take out sps and pps info.  



# Framework  
FFmpeg: most famous framework.  
libx265/ libx264: SW codec.  
libyuv: Google's open source YUV frame handling lib, better performance.  
OpenGL ES: Most of the video special effects, beautification algorithms.   
ExoPlayer：Support many protocols.  

# raspberry pi based Audio/ Video design  
1. Flash with pi OS  
Can flash with default pi tool.  

2. Setup QT env  
```bash
pi@raspberrypi:~ $ sudo apt-get update  
pi@raspberrypi:~ $ sudo apt-get install qt5-default  
pi@raspberrypi:~ $ sudo apt-get install qtcreator  
pi@raspberrypi:~ $ sudo apt-get install qtmultimedia5-dev  
pi@raspberrypi:~ $ sudo apt-get install libqt5serialport5-dev  
```
Then tools, options to configure the compiler.  

3. ffmpeg lib compile  
```bash
sudo apt-get install libomxil-bellagio-dev  
mv /home/pi/Downloads/x264-master.tar.bz2 ./  
tar xvf x264-master.tar.bz2   
cd x264-master/  
./configure --prefix=$PWD/_install  --enable-shared  
make && make install  
sudo cp _install/include /usr/ -rf  
sudo cp _install/lib /usr/ -rf  
# configure and install ffmpeg  
./configure --enable-shared --prefix=$PWD/_install --enable-gpl --enable-libx264 --enable-omx-rpi --enable-mmal --enable-hwaccel=h264_mmal --enable-decoder=h264_mmal --enable-encoder=h264_omx --enable-omx  
make && make install  
```

4. player software design  
In Android, it is written by Java or Kotlin, mostly we can use some open source ones.  
Like ExoPlayer (Kotlin/ Java), or ijkplayer (Java in Android).  
Video Decoder Sample design:  
    VideoDecoder (Audio/ Video Frame) -> AVSyncronizer (Audio/ Video Frame Queue) -> VideoPlayerController (Audio/ Video Output).  
Recorder design:  
    Audio: Input -> PCM queue -> Encode & Mux. Can also have output, like Oboe.  
    Video: Camera -> EGL Display / MediaCodec -> Mux. Can also have output, like video preview.  
    Mux: Combine Audio and Video, streams are aligned in time, most scenarios, Video will align with Audio, because humans are more sensitive to delays in audio.  

# Reference
https://www.videoproc.com/resource/h266-vvc.htm  
https://www.itu.int/rec/T-REC-H.266-202309-I/en  
https://ottverse.com/i-p-b-frames-idr-keyframes-differences-usecases/  
https://time.geekbang.org/column/intro/100021101  
https://time.geekbang.org/column/intro/100117601  
https://developer.android.com/reference/android/media/MediaRecorder  
https://developer.android.com/reference/android/media/MediaPlayer  
https://github.com/google/ExoPlayer  
https://github.com/bilibili/ijkplayer  
https://www.shadertoy.com/  
http://www.ffmpeg.org/download.html  
https://www.videolan.org/developers/x264.html  
https://xie.infoq.cn/article/c237112048d737b22162514eb  