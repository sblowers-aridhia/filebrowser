<template>
  <div
    @click="focus"
    class="shell"
    ref="scrollable"
    :class="{ ['shell--hidden']: !showShell }"
  >
    <div key="index-help" class="shell__result">
      <pre class="shell__text">{{ shellIntro }}</pre>
    </div>

    <div v-for="(c, index) in content" :key="index" class="shell__result">
      <div class="shell__prompt">
        <i class="material-icons">chevron_right</i>
      </div>
      <pre class="shell__text">{{ c.text }}</pre>
    </div>

    <div class="shell__result" :class="{ 'shell__result--hidden': !canInput }">
      <div class="shell__prompt">
        <i class="material-icons">chevron_right</i>
      </div>
      <pre
        tabindex="0"
        ref="input"
        class="shell__text"
        contenteditable="true"
        @keydown.prevent.38="historyUp"
        @keydown.prevent.40="historyDown"
        @keypress.prevent.enter="submit"
      />
    </div>
  </div>
</template>

<script>
import { mapMutations, mapState, mapGetters } from "vuex";
import { commands } from "@/api";
import {shellIntro, shellHelp} from "@/utils/shellHelp"

export default {
  name: "shell",
  computed: {
    ...mapState(["user", "showShell"]),
    ...mapGetters(["isFiles", "isLogged"]),
    path: function () {
      if (this.isFiles) {
        return this.$route.path;
      }

      return "";
    },
  },
  data: () => ({
    content: [],
    history: [],
    historyPos: 0,
    canInput: true,
    shellIntro: shellIntro,
  }),
  methods: {
    ...mapMutations(["toggleShell"]),
    scroll: function () {
      this.$refs.scrollable.scrollTop = this.$refs.scrollable.scrollHeight;
    },
    focus: function () {
      this.$refs.input.focus();
    },
    historyUp() {
      if (this.historyPos > 0) {
        this.$refs.input.innerText = this.history[--this.historyPos];
        this.focus();
      }
    },
    historyDown() {
      if (this.historyPos >= 0 && this.historyPos < this.history.length - 1) {
        this.$refs.input.innerText = this.history[++this.historyPos];
        this.focus();
      } else {
        this.historyPos = this.history.length;
        this.$refs.input.innerText = "";
      }
    },
    submit: function (event) {
      const cmd = event.target.innerText.trim();

      if (cmd === "") {
        return;
      }

      if (cmd === "clear" || cmd.startsWith("clear ")) {
        this.content = [];
        event.target.innerHTML = "";
        return;
      }

      if (cmd === "exit" || cmd.startsWith("exit ")) {
        this.content = [];
        event.target.innerHTML = "";
        this.toggleShell();
        return;
      }

      this.canInput = false;
      event.target.innerHTML = "";

      let results = {
        text: `${cmd}\n\n`,
      };

      this.history.push(cmd);
      this.historyPos = this.history.length;
      this.content.push(results);

      if (cmd==="help" || cmd.startsWith("help ")) {
        commands(
          this.path,
          cmd,
          () => {
            results.text += shellHelp;
            this.scroll();
          },
          () => {
            results.text = results.text.trimEnd();
            this.canInput = true;
            this.$refs.input.focus();
            this.scroll();
          }
        );
      } else if (cmd==="cd" || cmd.startsWith("cd ")) {
        commands(
          this.path,
          cmd,
          () => {
            results.text += `cd functionality is not possible within this shell terminal. Please navigate folders via the interface.`;
            this.scroll();
          },
          () => {
            results.text = results.text.trimEnd();
            this.canInput = true;
            this.$refs.input.focus();
            this.scroll();
          }
        );

      } else {

        commands(
          this.path,
          cmd,
          (event) => {
            results.text += `${event.data}\n`;
            this.scroll();
          },
          () => {
            results.text = results.text.trimEnd();
            this.canInput = true;
            this.$refs.input.focus();
            this.scroll();
          }
        );
      }
    },
  },
};
</script>
