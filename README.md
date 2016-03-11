# Android TextureView OpenGL ES 2.0 Example
It's a very simple example of using OpenGL ES 2.0 on Android platform. It uses TextureView for good integration with 
system. The example written in Kotlin and contains about 80 lines of code (with context creation, etc). So it's very 
simple sample for introducing in OpenGL in Android. If it's too complex for you - you can see [a bit easier example](https://github.com/egslava/example-android-opengl-textureview-easy).

It has been tested with and without "Don't keep activities" mode on Nexus 5.  The main code is here:
```Kotlin
class MainActivity : AppCompatActivity(), TextureView.SurfaceTextureListener {

    lateinit var renderer: RendererThread

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        texture_view.surfaceTextureListener = this
    }

    override fun onSurfaceTextureUpdated(surface: SurfaceTexture?) {} // called every time when swapBuffers is called
    override fun onSurfaceTextureSizeChanged(surface: SurfaceTexture?, width: Int, height: Int) {}
    override fun onSurfaceTextureDestroyed(surface: SurfaceTexture): Boolean{
        renderer.isStopped = true
        return false                // surface.release() manually, after the last render
    }
    override fun onSurfaceTextureAvailable(surface: SurfaceTexture, width: Int, height: Int) {
        renderer = RendererThread(surface)
        renderer.start()
    }

    class RendererThread(val surface: SurfaceTexture) : Thread() {

        var isStopped = false

        val config = intArrayOf(
            EGL_RENDERABLE_TYPE, EGL_OPENGL_ES2_BIT,
            EGL_RED_SIZE, 8,
            EGL_GREEN_SIZE, 8,
            EGL_BLUE_SIZE, 8,
            EGL_ALPHA_SIZE, 8,
            EGL_DEPTH_SIZE, 0,
            EGL_STENCIL_SIZE, 0,
            EGL_NONE
        )

        fun  chooseEglConfig(egl: EGL10, eglDisplay: EGLDisplay) : EGLConfig {
            val configsCount = intArrayOf(0);
            val configs = arrayOfNulls<EGLConfig>(1);
            egl.eglChooseConfig(eglDisplay, config, configs, 1, configsCount)
            return configs[0]!!
        }

        override fun run() {
            super.run()

            val egl = EGLContext.getEGL() as EGL10
            val eglDisplay = egl.eglGetDisplay(EGL_DEFAULT_DISPLAY)
            egl.eglInitialize(eglDisplay, intArrayOf(0, 0))   // getting OpenGL ES 2
            val eglConfig = chooseEglConfig(egl, eglDisplay);
            val eglContext = egl.eglCreateContext(eglDisplay, eglConfig, EGL_NO_CONTEXT, intArrayOf(EGL_CONTEXT_CLIENT_VERSION, 2, EGL_NONE));
            val eglSurface = egl.eglCreateWindowSurface(eglDisplay, eglConfig, surface, null)

            var colorVelocity = 0.01f
            var color = 0f
            while (!isStopped && egl.eglGetError() == EGL_SUCCESS) {
                egl.eglMakeCurrent(eglDisplay, eglSurface, eglSurface, eglContext)
                if (color > 1 || color < 0) colorVelocity *= -1
                color += colorVelocity

                GLES20.glClearColor(color / 2, color, color, 1.0f)
                GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT)
                egl.eglSwapBuffers(eglDisplay, eglSurface)

                Thread.sleep((1f / 60f * 1000f).toLong()) // in real life this sleep is more complicated
            }

            surface.release()
            egl.eglDestroyContext(eglDisplay, eglContext)
            egl.eglDestroySurface(eglDisplay, eglSurface)
        }
    }
}
```
Have fun! :-)
