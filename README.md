// MainActivity.kt
package com.example.jarvisai

import android.content.Intent
import android.content.pm.PackageManager
import android.os.Bundle
import android.speech.RecognizerIntent
import android.speech.tts.TextToSpeech
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import java.util.*

class MainActivity : AppCompatActivity(), TextToSpeech.OnInitListener {

    private lateinit var tts: TextToSpeech

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        tts = TextToSpeech(this, this)

        // App शुरू होते ही voice command सुनना
        startVoiceCommand()
    }

    // Text to Speech initialization
    override fun onInit(status: Int) {
        if (status == TextToSpeech.SUCCESS) {
            tts.language = Locale.US
            speak("Hello Yash, your Jarvis is active. Which app should I open?")
        }
    }

    private fun speak(text: String) {
        tts.speak(text, TextToSpeech.QUEUE_FLUSH, null, "")
    }

    private fun startVoiceCommand() {
        val intent = Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH)
        intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL, RecognizerIntent.LANGUAGE_MODEL_FREE_FORM)
        intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE, Locale.getDefault())
        startActivityForResult(intent, 100)
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)

        if (requestCode == 100 && resultCode == RESULT_OK) {
            val result = data?.getStringArrayListExtra(RecognizerIntent.EXTRA_RESULTS)
            val command = result?.get(0)?.toLowerCase(Locale.ROOT)

            if (command != null) {
                when {
                    command.contains("time") -> {
                        val currentTime = java.text.DateFormat.getTimeInstance().format(Date())
                        speak("The time is $currentTime")
                    }

                    command.contains("open") -> {
                        val appName = command.replace("open", "").trim()
                        openApp(appName)
                    }

                    command.contains("hello") -> {
                        speak("Hello Yash, how can I help you?")
                    }

                    command.contains("exit") || command.contains("stop") -> {
                        speak("Okay shutting down Jarvis")
                        finish()
                    }

                    else -> {
                        speak("Sorry, I did not understand the command")
                    }
                }
            }
        }
    }

    private fun openApp(appName: String) {
        val pm: PackageManager = packageManager
        val packages = pm.getInstalledApplications(0)

        for (packageInfo in packages) {
            val label = pm.getApplicationLabel(packageInfo).toString().toLowerCase(Locale.ROOT)
            if (label.contains(appName)) {
                val launchIntent = pm.getLaunchIntentForPackage(packageInfo.packageName)
                if (launchIntent != null) {
                    speak("Opening $appName")
                    startActivity(launchIntent)
                    return
                }
            }
        }

        speak("App not found: $appName")
        Toast.makeText(this, "App not found: $appName", Toast.LENGTH_SHORT).show()
    }

    override fun onDestroy() {
        if (::tts.isInitialized) {
            tts.stop()
            tts.shutdown()
        }
        super.onDestroy()
    }
}
