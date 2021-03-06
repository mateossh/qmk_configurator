<template>
  <div>
    <div ref="console">
      <controllerTop v-if="appInitialized" />
      <statusPanel />
      <controllerBottom />
    </div>
    <div class="hint hint-right">
      <a target="_blank" href="https://github.com/qmk/qmk_toolbox/releases">
        {{ $t('downloadToolbox.label') }}
      </a>
    </div>
    <div class="split-content">
      <div class="left-side">
        <layerControl />
      </div>
      <div class="right-side">
        <div class="keymap--area">
          <label class="keymap--label" :title="$t('ColorwayTip.title')">
            {{ $t('keymap.label') }}:
            <font-awesome-icon
              v-if="continuousInput"
              icon="keyboard"
              fixed-width
            />
          </label>
          &nbsp;
          <!-- maintain spacing for paragraph -->
          <select
            class="keymap--keyset"
            id="colorway-select"
            v-model="curIndex"
          >
            <option
              class="option"
              v-for="(name, index) in displayColorways"
              :key="index"
              :value="index"
              >{{ name }}</option
            >
          </select>
          <a
            id="favorite-colorway"
            :title="$t('favoriteColor')"
            @click="favColor"
            :class="{
              active: isFavoriteColor
            }"
          >
            <font-awesome-icon icon="star" size="lg" fixed-width />
          </a>
        </div>
        <visualKeymap :profile="false" />
        <span class="keymap--count"
          ><span class="keymap--counter">{{ keyCount }}</span
          >Keys</span
        >
      </div>
    </div>
  </div>
</template>

<script>
import capitalize from 'lodash/capitalize';
import {
  mapMutations as _mapMutations,
  createNamespacedHelpers,
  mapState as _mapState,
  mapGetters as _mapGetters,
  mapActions
} from 'vuex';
const { mapState, mapGetters, mapMutations } = createNamespacedHelpers(
  'keymap'
);
import ControllerTop from '@/components/ControllerTop';
import StatusPanel from '@/components/StatusPanel';
import ControllerBottom from '@/components/ControllerBottom';
import VisualKeymap from '@/components/VisualKeymap';
import LayerControl from '@/components/LayerControl';
import * as jquery from '@/jquery';

export default {
  name: 'Main',
  props: {},
  components: {
    ControllerTop,
    StatusPanel,
    ControllerBottom,
    VisualKeymap,
    LayerControl
  },
  computed: {
    ..._mapState('app', ['appInitialized', 'configuratorSettings']),
    ..._mapGetters('app', ['keyCount']),
    ...mapState(['continuousInput']),
    ...mapGetters(['colorwayIndex', 'colorways', 'size']),
    curIndex: {
      get() {
        return this.colorwayIndex;
      },
      set(value) {
        this.nextColorway(value);
      }
    },
    displayColorways() {
      return this.colorways.map(keyset => {
        return keyset
          .replace(/-/g, ' ')
          .split(' ')
          .map(word => capitalize(word))
          .join(' ')
          .replace(/Gmk/, 'GMK')
          .replace(/^Sa/, 'SA')
          .replace(/^Dsa/, 'DSA')
          .replace(/Wob/, 'WOB')
          .replace(/Ta/, 'TA');
      });
    },
    redditPost() {
      return 'https://www.reddit.com/r/MechanicalKeyboards/comments/aio97b/qmk_configurator_updates_beta_need_your_input/';
    },
    isFavoriteColor() {
      return (
        this.configuratorSettings.favoriteColor &&
        this.displayColorways[this.curIndex].toLowerCase() ===
          this.configuratorSettings.favoriteColor.toLowerCase()
      );
    }
  },
  methods: {
    ...mapActions('app', ['setFavoriteColor']),
    ...mapMutations(['nextColorway']),
    ..._mapMutations('app', ['resetListener']),
    favColor() {
      if (this.isFavoriteColor) {
        this.setFavoriteColor('');
      } else {
        this.setFavoriteColor(this.displayColorways[this.curIndex]);
      }
    }
  },
  mounted() {
    jquery.init();
    // Loading favorite color
    if (this.configuratorSettings.favoriteColor) {
      const favoriteColor = this.configuratorSettings.favoriteColor.toLowerCase();
      this.curIndex = this.displayColorways.findIndex(
        color => color.toLowerCase() === favoriteColor
      );
    }
  },
  beforeDestroy() {
    this.resetListener();
  }
};
</script>
<style>
.hint-right {
  display: grid;
  justify-content: end;
}
#colorway-select {
  font-family: 'Roboto', 'Helvetica Neue', Helvetica, Arial, sans-serif;
}
.beta-feedback {
  position: fixed;
  right: 10px;
  top: 30px;
}
.beta-button {
  height: 30px;
  font-size: 15px;
  border-radius: 9px;
  cursor: pointer;
}
.keymap--label {
  float: left;
}
.keymap--counter {
  display: inline-block;
  padding: 0 5px;
  margin-top: 2px;
}
.keymap--count {
  float: right;
  color: #999;
}
.keymap--keyset {
  float: right;
}
.keymap--area {
  margin-top: 1em;
  margin-bottom: 1em;
  height: 1.5em;
}
</style>
