<template>
  <div class="navigation">
    <div class="tabs" ref="tabsContainer">
      <div class="tabs-inner">
        <div v-for="tab in tabs" :key="tab.path" class="tab" @click="switchTab(tab.path)"
          :class="{ active: activeTab === tab.path }">
          {{ tab.name }}
        </div>
      </div>
    </div>
  </div>
</template>



<script>
import { shallowRef, reactive, onMounted, computed } from 'vue';
import Hammer from 'hammerjs';
import { useStore } from 'vuex'; // Importiere das useStore-Hook
// eslint-disable-next-line
import { iconColor } from '../components/farben';

export default {
  name: 'SwipeNavigation',
  props: ['onTabSwitch'],
  setup(props) {
    const store = useStore();
    // eslint-disable-next-line
    const currentUser = computed(() => store.state.currentUser);
    const tabs = reactive([
      { name: 'Public', path: '/public' },
      { name: 'Personal', path: '/fandf' },
    ]);

    // Versuche, den aktiven Tab aus dem Local Storage zu laden oder setze den ersten Tab als Standard
    const activeTab = shallowRef(sessionStorage.getItem('activeTab') || tabs[0].path);

    const tabIndexes = tabs.reduce((acc, tab, i) => ({ ...acc, [tab.path]: i }), {});

    const switchTab = (path) => {
      activeTab.value = path;
      sessionStorage.setItem('activeTab', path); // Speichere den aktiven Tab im Local Storage
      props.onTabSwitch(path);
    };

    const switchToTabByIndex = (index) => {
      if (index < 0 || index >= tabs.length) return;
      switchTab(tabs[index].path);
    };

    const nextTab = () => {
      const currentIndex = tabIndexes[activeTab.value];
      switchToTabByIndex(currentIndex + 1);
    };

    const previousTab = () => {
      const currentIndex = tabIndexes[activeTab.value];
      switchToTabByIndex(currentIndex - 1);
    };

    onMounted(() => {
      const el = document;
      const hammer = new Hammer(el);
      hammer.get('swipe').set({ direction: Hammer.DIRECTION_HORIZONTAL });
      hammer.on('swiperight', previousTab);
      hammer.on('swipeleft', nextTab);

      // Dieses Beispiel nimmt an, dass `currentUser.value.farbe` bereits gesetzt ist.
      // Andernfalls müsste hier eine entsprechende Überprüfung erfolgen.

    });

    return { activeTab, nextTab, previousTab, switchTab, tabs };
  },
};
</script>

 

<style scoped>
.tabs {
  display: flex;
  justify-content: center;
  /* Center the tabs horizontally */
  overflow-x: auto;
  padding: 10px;
  background: #ffffff;
  border-bottom: 1px solid #e1e4e8;
}

.tabs-inner {
  display: flex;
  gap: 20px;
  min-width: 100%;
  justify-content: space-evenly;
  /* Ensure equal spacing between tabs */
}

.tab {
  padding: 10px 20px;
  font-size: 16px;
  color: #333;
  text-align: center;
  flex-shrink: 0;
  cursor: pointer;
  transition: all 0.3s ease;
  position: relative;
}

.tab::after {
  content: "";
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 2px;
  background-color: var(--iconColor);
  transform: scaleX(0);
  transform-origin: left;
  transition: transform 0.3s ease;
}

.tab.active {
  color: var(--iconColor);
}

.tab.active::after {
  transform: scaleX(1);
}
</style>
