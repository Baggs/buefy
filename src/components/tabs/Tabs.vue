<template>
    <div class="b-tabs" :class="{ 'is-fullwidth': expanded }">
        <nav class="tabs" :class="navClasses">
            <ul>
                <li
                    v-for="(tabItem, index) in tabItems"
                    :key="index"
                    v-show="tabItem.visible"
                    :class="{ 'is-active': activeTab === index, 'is-disabled': tabItem.disabled }">
                    <a @click="tabClick(index)">
                        <template v-if="tabItem.$slots.header">
                            <b-slot-component
                                :component="tabItem"
                                name="header"
                                tag="span" />
                        </template>
                        <template v-else>
                            <b-icon
                                v-if="tabItem.icon"
                                :icon="tabItem.icon"
                                :pack="tabItem.iconPack"
                                :size="size"/>
                            <span>{{ tabItem.label }}</span>
                        </template>
                    </a>
                </li>
            </ul>
        </nav>
        <section class="tab-content" :class="{'is-transitioning': isTransitioning}">
            <transition
                :name="transitionName"
                @before-enter="isTransitioning = true"
                @after-enter="isTransitioning = false">
                <slot/>
            </transition>
        </section>
    </div>
</template>

<script>
    import Icon from '../icon/Icon'
    import SlotComponent from '../../utils/SlotComponent'

    export default {
        name: 'BTabs',
        components: {
            [Icon.name]: Icon,
            [SlotComponent.name]: SlotComponent
        },
        props: {
            value: [String, Number],
            expanded: Boolean,
            type: String,
            size: String,
            position: String,
            animated: {
                type: Boolean,
                default: true
            }
        },
        data() {
            return {
                activeTab: this.value || 0,
                tabItems: [],
                contentHeight: 0,
                _isTabs: true, // Used internally by TabItem
                isTransitioning: false
            }
        },
        computed: {
            navClasses() {
                return [
                    this.type,
                    this.size,
                    this.position,
                    {
                        'is-fullwidth': this.expanded,
                        'is-toggle-rounded is-toggle': this.type === 'is-toggle-rounded'
                    }
                ]
            }
        },
        watch: {
            /**
             * When v-model is changed set the new active tab.
             */
            value(value) {
                this.changeTab(value)
            },

            /**
             * When tab-items are updated, set active one.
             */
            tabItems() {
                if (this.tabItems.length) {
                    this.tabItems[this.activeTab].isActive = true
                }
            }
        },
        methods: {
            /**
             * Change the active tab and emit change event.
             */
            changeTab(newIndex) {
                if (this.activeTab === newIndex) return

                this.deactivate(this.activeTab, newIndex)
                this.activate(this.activeTab, newIndex)
                this.activeTab = newIndex
                this.$emit('change', newIndex)
            },

            /**
             * Tab click listener, emit input event and change active tab.
             */
            tabClick(value) {
                this.$emit('input', value)
                this.changeTab(value)
            },

             /**
             * Activate tab, alter animation name based on the index.
             */
            activate(oldIndex, index) {
                if (!this.animated) {
                    this.transitionName = null
                } else {
                    this.transitionName = index < oldIndex
                        ? 'slide-next'
                        : 'slide-prev'
                }
                this.tabItems[index].isActive = true
            },

            /**
             * Deactivate tab, alter animation name based on the index.
             */
            deactivate(oldIndex, index) {
                if (!this.animated) {
                    this.transitionName = null
                } else {
                    this.transitionName = index < oldIndex
                        ? 'slide-next'
                        : 'slide-prev'
                }
                this.tabItems[oldIndex].isActive = false
            }
        },
        mounted() {
            if (this.tabItems.length) {
                this.tabItems[this.activeTab].isActive = true
            }
        }
    }
</script>
