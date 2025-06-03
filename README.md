# Polyphonical-musical-synthesizer
Final project for the subject digital speech and audio project, this is basically a MIDI alternative in some sense


make release
As a written report for the practice, complete, in this same document and using Markdown format, the exercises indicated below.

Exercises
ADSR Envelope
Using a simple instrument model (you may use InstrumentDumb), generate four instruments that allow visualizing the behavior of an ADSR envelope curve:

Generic ADSR instrument
An instrument with a generic ADSR envelope, where each of its parameters—Attack (A), Decay (D), Sustain (S), and Release (R)—can be clearly observed.

Percussive instrument (e.g., guitar or piano)
In this instrument, the sound should have a fast attack, no sustain, and a slow release. For this type of instrument, there are two possible scenarios:

The performer holds the note until it fully decays on its own.

The performer releases the note before it fully decays, initiating an abrupt decrease of the sound to its end.
You must illustrate both possible note endings in this report.

“Bowed” or “wind-like” instrument (e.g., bowed strings such as violins, or some wind instruments)
In these instruments, the attack is relatively fast until reaching the sustain level (without overshoot), and the release is also quite rapid.

For all four cases, you must include a plot in which the ADSR curve is clearly visible. You should add any necessary information for its correct interpretation; however, that information can be reduced to placing appropriate labels and titles directly on each graph (this alternative will be positively evaluated).

After being inspired by “dumb.orc,” we generated four .orc files based on the problem statement specifications: generico.orc, percusivo1.orc, percusivo2.orc, and plano.orc. Next, since we already have doremi.sco, we applied it to the four instruments to easily observe the differences, obtaining generico.wav, percusivo1.wav, percusivo2.wav, and plano.wav after running synth. Then, we generated a short, precise score (.sco) to analyze the sounds more accurately and produced new .wav files (generico_def.wav, percusivo1_def.wav, percusivo2_def.wav, and plano_def.wav), which we used for the plots below.



Dumb and Sine Instruments
Implement a Sine instrument using InstrumentDumb as a model. The signal must be formed by looking up values in a table.

Below, include the code of the file seno.cpp with the methods of the Seno class.

In our case, the file is named instrument_sin.cpp.


#include "instrument_sin.h"
#include <math.h>
#include <iostream>

using namespace std;
using namespace upc;

InstrumentSin::InstrumentSin(const std::string &params)
: adsr(SamplingRate, params) {
    x.resize(BSIZE);
    bActive = false;

    // Create sine table of N samples
    KeyValue kv(params);               // The parameters defined in dumb.orc
    int N;
    if (!kv.to_int("N", N))            // If the file did not include parameter N
        N = 40;
    tbl.resize(N);
    float phase = 0;
    float stepTbl = 2 * 3.1415926f / (float) N;
    for (int i = 0; i < N; ++i) {
        tbl[i] = sin(phase);
        phase += stepTbl;
    }
    id = 0;
}

void InstrumentSin::command(long cmd, long note, long vel) {
    if (cmd == 9) {          // Note On
        bActive = true;
        adsr.start();
        // Compute step based on MIDI note
        float freq = pow(2.0f, (note - 69.0f) / 12.0f) * 440.0f;
        step = freq * 2.0f * 3.1415926f / SamplingRate;
        A = vel / 127.0f;
        phaseIndex = 0;
        id = 0;
    }
    else if (cmd == 8) {     // Note Off
        adsr.stop();
    }
    else if (cmd == 0) {     // All notes off
        adsr.end();
    }
}

const std::vector<float>& InstrumentSin::synthesize() {
    // If ADSR has finished, return silence and deactivate voice
    if (!adsr.active()) {
        x.assign(x.size(), 0.0f);
        bActive = false;
        return x;
    }
    else if (!bActive) {
        // If voice not yet active, return previous buffer (silence or release tail)
        // but allow ADSR to process the release phase
        return x;
    }

    // Fill buffer with interpolated sine samples
    for (unsigned int i = 0; i < x.size(); ++i) {
        phaseIndex += step;
        while (phaseIndex > 2 * 3.1415926f) {
            phaseIndex -= 2 * 3.1415926f;
        }
        id = (int)(phaseIndex / (2 * 3.1415926f) * tbl.size());
        x[i] = A * tbl[id];
    }

    // Apply ADSR envelope to entire buffer
    adsr(x);

    return x;
}
Explain the method used to assign a value to the signal from the table contents, and include a plot in which both the table values and the generated signal values are clearly visible (use dots instead of lines).
We store one period of the sinusoidal signal in a lookup table with N samples. Then, by reading this stored period faster or slower, we generate higher or lower pitches as needed.

If you have implemented table lookup synthesis using an external file, include below the code of the command() method.

(N/A in our implementation, since we generated the table in code rather than loading from an external file.)

Sound Effects
Include two plots in which the effects of tremolo and vibrato on a sinusoidal signal are clearly visible. You must explain in detail how the effect parameters (modulation frequency and modulation index) manifest in the generated signal (it will be positively evaluated if the explanation is contained within the plots themselves, without needing additional text).

Sinusoidal signal with tremolo:
– Modulation frequency fm = 20 Hz and amplitude modulation index A = 0.5



Sinusoidal signal with vibrato:
– Modulation frequency fm = 10 Hz and modulation index I = 0.4



We generated a .sco file configured so that the effects are applied to middle C (MIDI 60).

If you have created any custom effect, explain what it consists of, how you implemented it, and what result it produced. Include, in the work/ejemplos directory, the files necessary to appreciate the effect, and indicate below the command needed to generate the audio files using the synth program.

We applied clipping, meaning that when the signal exceeds a threshold, it saturates and does not go beyond that value. This is clearly seen in the maximum amplitude of the signal (which we can find at the top-left margin) if we allow 100% (clipping_out), 80% (clipping80), or 50% (clipping50). Indeed, 6748 is half of 13496. We observe that as we progress through the three cases, we lose signal nuances in the upper portion.





FM Synthesis
Construct an FM synthesis instrument according to the explanations in the problem statement and the article by John M. Chowning (https://web.eecs.umich.edu/~fessler/course/100/misc/chowning-73-tso.pdf). The instrument should use as basic parameters the numbers N1, N2, and the modulation index I, which must be expressed in semitones.

Use the instrument to generate a vibrato with reasonable parameters and include a plot where the correspondence between the values N1, N2, and I and the obtained signal is clearly visible.

We demonstrate how “InstrumentFM” works through the following three plots:





We observe the requested vibrato in the following plot:



Regarding the requested correspondence—in somewhat less academic terms—I represents how powerful the modulation is. On the other hand, N1 and N2 indicate the ratio between the carrier frequency (f₀) and the modulation frequency (fₘ).

Use the instrument to generate a clarinet-like sound and a bell-like sound. Take the sound parameters (N1, N2, and I) and the ADSR envelope settings from the cited article. Using these sounds, generate two diatonic scales (doremi.sco) and place the resulting files in work/doremi/clarinete.wav and work/doremi/campana.wav.

We have added the relevant files to the indicated folder.

Optionally, you can upload other scales using interesting sounds—for example, strings, pianos, percussion, lightsabers from Star Wars, etc.—to the work/doremi directory.

We also added an acoustic piano, a violin, and a cello to the indicated folder, since—even though we are Star Wars fans—we considered those more realistic for this assignment.

Orchestration Using the synth Program
Use the synth program to generate songs from your MIDI score. At a minimum, you must include the orchestration of the song “You’ve Got a Friend in Me” (file ToyStory_A_Friend_in_me.sco) by the great Randy Newman (https://open.spotify.com/artist/3HQyFCFFfJO3KKBlUfZsyW/about).

In this arrangement, track 1 corresponds to the solo instrument (it may be piano, flutes, violins, etc.), and track 2 to the bass (electric bass, double bass, tuba, etc.).

Place the result, along with the necessary files to generate it, in the work/music directory.

Indicate below the command needed to generate the audio signal (assuming that all required files are in the indicated directory).

We created ToyStory_A_Friend_in_me.orc inspired by a saxophone and a tuba. Since we have the .sco file, we can generate ToyStory_A_Friend_in_me.wav.

Therefore, the required command is:

~/PAV/P5/work/music$ ~/PAV/bin/synth ToyStory_A_Friend_in_me.orc ToyStory_A_Friend_in_me.sco ToyStory_A_Friend_in_me.wav -g 0.1
You may also orchestrate more complex pieces, such as the Hawaii 5-0 soundtrack or John Lennon’s Christmas song Happy Xmas (War Is Over) (file The_Christmas_Song_Lennon.sco), or any other song of your choice. Instrumental richness, modeling, and the final result will be positively evaluated.

Place the generated files, together with their score, instrument, and effect files, into the work/music directory.

Indicate below the command needed to generate each signal using the different files.

Since the problem statement gives us freedom to choose pieces of our liking—and considering instrumental richness—each member of our pair chose a piece, which are detailed below.

We tried the piece “Fly Me to the Moon” by Frank Sinatra, downloaded in the piano version (thus, two voices: one for the left hand and one for the right hand). The command is analogous to the one above:


~/PAV/P5/work/music$ ~/PAV/bin/synth flymetothemoon.orc flymetothemoon.sco flymetothemoon.wav -g 0.1
Finally, we must highlight “Laplace’s Angel” by Will Wood, which has 37 tracks and combines different aspects we covered throughout the practice. Again, the command is analogous to the previous ones:


~/PAV/P5/work/music$ ~/PAV/bin/synth -b 150 laplacesangels.orc laplacesangels.sco laplacesangels.wav -g 0.1
NOTE:
Don’t forget to listen to the generated result and check that no unwanted noises or distortions occur. Above all, consider the hearing health of the person who will grade your work.
