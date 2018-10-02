# Outputting MIDI

## Creating a connection to the MIDI port

SuperCollider provides a class called `MIDIOut` to handle outputting MIDI to destinations outside SuperCollider. Before using `MIDIOut`, you will first need to call the `init` method of `MIDIClient`:

```
MIDIClient.init;
```

This will output to the post window a list of MIDI devices such as the following:

```
MIDI Sources:
	MIDIEndPoint("System", "Timer")
	MIDIEndPoint("System", "Announce")
	MIDIEndPoint("Midi Through", "Midi Through Port-0")
	MIDIEndPoint("Scarlett 6i6 USB", "Scarlett 6i6 USB MIDI 1")
	MIDIEndPoint("SuperCollider", "out0")
	MIDIEndPoint("SuperCollider", "out1")
MIDI Destinations:
	MIDIEndPoint("Midi Through", "Midi Through Port-0")
	MIDIEndPoint("Scarlett 6i6 USB", "Scarlett 6i6 USB MIDI 1")
	MIDIEndPoint("SuperCollider", "in0")
	MIDIEndPoint("SuperCollider", "in1")
```

I want to sound MIDI data out through my Scarlett audio interface, so I create a `MIDIClient` corresponding to one of those devices by copying and pasting the appropriate device names into a call to `MIDIOut.newByName()`:

```
~midiOut = MIDIOut.newByName("Scarlett 6i6 USB", "Scarlett 6i6 USB MIDI 1");
```

`MIDIOut` also has a property called `latency` which adds latency before sending MIDI messages, in order to match SuperCollider's internal latency. It's advisable to set this to the server's latency, and can be done at the time of creating the `MIDIOut`:

```
~midiOut = MIDIOut.newByName("Scarlett 6i6 USB", "Scarlett 6i6 USB MIDI 1").latency_(s.latency);
```

Or later if you prefer:

```
~midiOut.latency = s.latency;
```

You may occasionally find you need to tweak the latency value of your `MIDIOut` in order to get it to exactly match synths in SuperCollider. You can set the value at any time, including when a pattern is playing MIDI data.

## Sending data from the port

The class can be used in two different ways:
1. Calling methods on `MIDIOut` directly. This is a more direct approach.
1. Using `MIDIOut` in conjunction with patterns, involving a special type of `Event` called `\midi`.

We will initially examine the second way of working with `MIDIOut`, since it is common to write musical sequences in SuperCollider using patterns.

### Sending MIDI data by using patterns

Using patterns to control MIDI data is almost identical to using patterns to control SuperCollider `SynthDef`s. The event type you need to use is `\midi`. Here is an example:

```
(
Pdef(\myPattern,
	Pbind(
		\type,\midi,
		\midiout,~midiOut,
		\midicmd,\noteOn,
		\chan,0,
		\dur, 0.25,
		\degree, Pseq({7.rand2}!16,inf),
		\octave, 4
	)
).play;
)
```
A pattern which sends MIDI data uses some special keys:

`\midiout`: The `MIDIOut` instance you want to send MIDI data through (see above).
`\midicmd`: The type of MIDI message you want to send; in this case `\noteOn`.
`\chan`: The MIDI channel number. SuperCollider thinks of these as 0-based, so the MIDI channel numbers in your patterns should be between 0 (for MIDI channel 1) and 15 (for MIDI channel 16).

Note that as well as specifying MIDI note numbers directly using the key `\midinote`, you can also use `\note` or `\degree` and these will be converted to MIDI note numbers in the same way that they would be converted to frequency values in a pattern written to play a `SynthDef`. And of course, `\degree` can be used in conjunction with `\scale`.