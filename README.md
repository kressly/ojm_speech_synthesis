# ojm_speech_synthesis
Cross browsers speech synthesis for javascript that works on mobile and desktop and chromium-based browsers and Firefox. Many speech synthesis work in one browser but not in the other and the solutions out there are so difficult to implement so i decided to share my functions with you.

Just copy these and change it. u face challenges, contact me 


 <button id="speak-btn">Click to Speak</button>

<script>
    var lang_abbreviation = 'en';
    var maxPhraseLength = 200; // Set the maximum length for each phrase

    function ojm_speech_synthesis(message) 
	{
		//On applique le mem synthesis pour les chromes browsers.
    	const isChromium = /Chrome|Chromium/.test(navigator.userAgent) && /Google Inc/.test(navigator.vendor);

		if (isChromium) 
		{
			ojm_speack_now(message);
		} 
		else 
		{
			speak_fallback(message);
		}
	}


    function ojm_speack_now(text) 
    {
        try 
        {
            window.speechSynthesis.cancel();
            
            // Dummy phrase to initialize Google voices je met un point
            const dummyUtterance = new SpeechSynthesisUtterance('.');
            dummyUtterance.lang = lang_abbreviation === 'fr' ? 'fr-FR' : 'en-GB';
            window.speechSynthesis.speak(dummyUtterance);

            // Stop the dummy utterance after 1 second
            setTimeout(function() 
            {
                window.speechSynthesis.cancel();

                const phrases = splitTextIntoPhrases(text);

                phrases.forEach((phrase, index) => 
                {
                    const utterance = new SpeechSynthesisUtterance(phrase);
                    utterance.lang = lang_abbreviation === 'fr' ? 'fr-FR' : 'en-GB';
                    utterance.pitch = 1;
                    utterance.rate = 1;
                    utterance.volume = 1;

                    const voices = window.speechSynthesis.getVoices();

                    if (!voices.length) 
                    {
                        console.warn("No voices available. Using the default browser voice.");
                        utterance.voice = null; // Using null will default to the browser's voice
                    } 
                    else 
                    {
                        const selectedVoice = voices.find(voice => 
                            voice.name.includes('Google US English') || 
                            voice.name.includes('Google UK English') || 
                            voice.name.includes('Google français')
                        );

                        utterance.voice = selectedVoice || voices[0]; // Use the first voice if the selected one is not found
                    }

                    // Add an event listener to speak the next phrase after the current one ends
                    utterance.onend = function() 
                    {
                        if (index < phrases.length - 1) 
                        {
                            window.speechSynthesis.speak(new SpeechSynthesisUtterance(phrases[index + 1]));
                        }
                    };

                    window.speechSynthesis.speak(utterance);
                });
            }, 1000); // Wait for 1 second before canceling the dummy utterance and speaking the actual text.

            return true;
        } 
        catch (error) 
        {
            console.error('Speech synthesis error:', error);
            return false;
        }
    }

    function splitTextIntoPhrases(text) 
    {
        // Split text by sentence-ending punctuation (., !, ?)
        const sentenceEndings = /([.!?])/;
        const sentences = text.split(sentenceEndings);

        const phrases = [];
        let current_chunk = '';

        // Combine sentences into chunks, ensuring we don't break in the middle of a phrase
        for (let i = 0; i < sentences.length; i += 2) 
        {
            let sentence = sentences[i].trim() + (sentences[i + 1] ? sentences[i + 1] : '');

            // Check if adding this sentence exceeds the chunk size
            if ((current_chunk + ' ' + sentence).length <= maxPhraseLength) 
            {
                current_chunk += (current_chunk ? ' ' : '') + sentence;
            } 
            else 
            {
                phrases.push(current_chunk);
                current_chunk = sentence;
            }
        }

        // Add any remaining text as the last chunk
        if (current_chunk) 
        {
            phrases.push(current_chunk);
        }

        return phrases;
    }

    function speak_fallback(text) 
    {
        if ('speechSynthesis' in window) 
        {
            var synth = window.speechSynthesis;
            var msg = new SpeechSynthesisUtterance(text);
            msg.lang = lang_abbreviation === 'fr' ? 'fr-FR' : 'en-GB';
            msg.volume = 1;
            msg.rate = 1;

            synth.speak(msg);
        } 
        else 
        {
            console.warn("Speech synthesis not supported in this browser.");
        }
    }

    // Event listener for button click outside the function
    document.getElementById('speak-btn').addEventListener('click', function() 
    {
        const message = "Jesus is Lord. Here is a long message to test the speech synthesis function. It should be split into smaller phrases to avoid issues with long text. Make sure to test this functionality in your browser."; // Your long text
        
        ojm_speech_synthesis_play(message);
    });
</script>


<button id="pause-btn">Pause/Resume</button>

<script>
    var isPaused = false; // Track if the speech is currently paused

    function ojm_speech_synthesis_pause() 
    {
        if (!window.speechSynthesis.speaking) 
        {
            console.warn("No speech is currently playing.");
            return;
        }

        if (isPaused) 
        {
            // If already paused, resume the speech
            window.speechSynthesis.resume();
            console.log("Speech resumed.");
        } 
        else 
        {
            // If not paused, pause the speech
            window.speechSynthesis.pause();
            console.log("Speech paused.");
        }

        isPaused = !isPaused; // Toggle the pause/resume state
    }

    // Event listener for pause/resume button
    document.getElementById('pause-btn').addEventListener('click', function() 
    {
        ojm_speech_synthesis_pause();
    });
</script>


<button id="stop-btn">Stop</button>

<script>
    function ojm_speech_synthesis_stop() 
    {
        // Check if speech synthesis is currently speaking
        if (window.speechSynthesis.speaking || window.speechSynthesis.paused) 
        {
            // Cancel any ongoing speech
            window.speechSynthesis.cancel();
            console.log("Speech stopped.");
        } 
        else 
        {
            console.warn("No speech is currently playing.");
        }
    }

    // Event listener for stop button
    document.getElementById('stop-btn').addEventListener('click', function() 
    {
        ojm_speech_synthesis_stop();
    });
</script>


<button id="loop-btn">Loop</button>

<button id="stop-loop-btn">Stop Loop</button>


<script>
    let isLooping = false;
    let currentUtterance;

    function ojm_speech_synthesis_loop(message) 
    {
        if (isLooping) 
        {
            console.log("Looping is already active.");
            return;
        }

        isLooping = true;
        currentUtterance = new SpeechSynthesisUtterance(message);
        
        // Set up event listener for when the utterance ends
        currentUtterance.onend = function() 
        {
            // Speak the utterance again for looping
            if (isLooping) 
            {
                window.speechSynthesis.speak(currentUtterance);
            }
        };

        // Start speaking the utterance
        window.speechSynthesis.speak(currentUtterance);
        console.log("Started looping speech.");
    }

    // Event listener for loop button
    document.getElementById('loop-btn').addEventListener('click', function() 
    {
        const message = "Jesus is Lord. Here is a message that will loop continuously."; // Your message to loop
        ojm_speech_synthesis_loop(message);
    });

    // Function to stop looping
    function ojm_speech_synthesis_stop_loop() 
    {
        if (isLooping) 
        {
            isLooping = false;
            window.speechSynthesis.cancel(); // Stop the current utterance
            console.log("Looping stopped.");
        } 
        else 
        {
            console.warn("Looping is not currently active.");
        }
    }

    // You can add a button to stop the loop
    document.getElementById('stop-loop-btn').addEventListener('click', function() 
    {
        ojm_speech_synthesis_stop_loop();
    });
</script>


