<template>
  <input type="file" @change="parseFile" />
  <h1>{{ song.name }}</h1>
  {{ song.duration }}

  <div id="draw"></div>
</template>

<script>
import { Midi } from "@tonejs/midi";
import { SVG } from "@svgdotjs/svg.js";
import hslToHex from "@d85/hsl-to-hex";
import { ref, onMounted, watch, reactive } from "vue";
export default {
  name: "midi",
  setup() {
    const song = reactive({
      name: "unnamed song",
      header: {},
      tracks: [],
    });
    const draw = SVG();
    onMounted(() => {
      draw.addTo("#draw");
    });

    function parseFile(event) {
      const reader = new FileReader();
      reader.readAsArrayBuffer(event.target.files[0]);
      reader.onload = () => {
        parseSong(new Midi(reader.result));
      };
    }

    function parseSong(midi) {
      song.name = midi?.header?.name || "unnamed song loaded";
      song.duration = getDuration(midi.tracks);
      song.header = midi.header;
      for (let track of midi.tracks) {
        if (track.channel && track.notes.length > 0) {
          song.tracks.push(track);
        }
      }
      drawSong(song);
    }

    function drawSong(s) {
      draw.size("100%", 800);
      draw.viewbox(0, 0, s.tracks.length * 20000, s.duration);
      s.tracks.forEach((track, i) => {
        let trackGroup = draw.group();
        console.log(i);
        for (let note of track.notes) {
          let rect = trackGroup.rect(note.durationTicks, 1000);
          rect.fill(getColor(note));
          rect
            .move(note.ticks, i * 10000)
            .dmove(0, ((note.midi - 9) % 12) * 500);
        }
      });
    }

    function getDuration(tracks) {
      let total = 0;
      for (let track of tracks) {
        let count = track.notes.length;
        if (count < 1) continue;
        let last = track.notes[count - 1];
        let dur = last.ticks + last.durationTicks;
        if (dur > total) {
          total = dur;
        }
      }
      return total;
    }

    function getColor(note) {
      let hue = ((note.midi - 9) % 12) * 30;
      let color = hslToHex(hue, 100, 50);
      return color;
    }

    return {
      parseFile,
      song,
    };
  },
};
</script>
