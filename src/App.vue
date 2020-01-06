<template>
	<div class="Menu">

        <MenuBurger :handleBurgerClicked="clickBurger" />

        <MenuShadow :isActive="menuActivator" :handleShadowClicked="clickShadow" />

	    <div class="Menu__panel-wrapper"
            :class="{'isActive': menuActivator}"
            :style="[style_wrapperStyle, menuActivator ? style_wrapperActiveStyle : {}]"
        >
            <div v-if="user.loggedIn" style="display:flex;padding-top:5px;background-color:#ffffff;margin-bottom:0px;height:110px;margin:auto;padding-left:10px;">
                <div style="display:inline-flex">
                    <img v-if="user && user.avatar" :src="user.avatar" :alt="user.username">
                    <img v-else src="https://avataaars.io/?avatarStyle=Circle&topType=Hat&accessoriesType=Blank&facialHairType=BeardMedium&facialHairColor=BrownDark&clotheType=BlazerShirt&eyeType=Default&eyebrowType=Default&mouthType=Default&skinColor=Light" alt="avatar" style="width:100px;height:100px;margin-right:10px;">
                    <div style="display:flex!important;flex-direction:column;">
                        <p style="margin-bottom:0px" class="subtitle">{{ user.username }}</p>
                        <p>{{ user.role }}</p>
                        <div style="display:flex!important;flex-direction:row;">
                            <a href="#" class="btn btn-danger">Logout</a>
                            <a href="#" class="btn btn-primary">Profile</a>
                        </div>
                    </div>
                </div>
            </div>
            <div style="position: relative">
	        <!-- prev -->
            <MenuPanel
                :list="content_prevItem"
                :functionalityStyle="style_panelStyle"
                :positionStyle="panel_prevPositionStyle"
                :isTranslating="isTranslating"
                :transitionStyle="style_transitionStyle"
                :showHeaderArrow="prevItemHasParent"
            />

	        <!-- staging -->
            <MenuPanel
                :list="content_currentItem"
                :functionalityStyle="style_panelStyle"
                :positionStyle="panel_stagingPositionStyle"
                :isTranslating="isTranslating"
                :transitionStyle="style_transitionStyle"
                :showHeaderArrow="currentItemHasParent"
                :handleHeaderClicked="clickPrevItem"
                :handleItemClicked="clickNextItem"
            />

            <!-- next -->
            <MenuPanel
                :list="content_nextItem"
                :functionalityStyle="style_panelStyle"
                :positionStyle="panel_nextPositionStyle"
                :isTranslating="isTranslating"
                :transitionStyle="style_transitionStyle"
                :showHeaderArrow="true"
            />
            </div>
            <div style="position:relative;">
                <p>foot</p>
            </div>
	    </div>
	</div>
</template>

<script>
import demoData from './demo-data.js';

import functionalityStyle from './mixins/functionalityStyle.mixin';
import panelControl from './mixins/panelControl.mixin';
import contentControl from './mixins/contentControl.mixin';

import RightArrowIcon from './icons/RightArrowIcon.vue';
import LeftArrowIcon from './icons/LeftArrowIcon.vue';
import MenuBurger from './components/MenuBurger.vue';
import MenuShadow from './components/MenuShadow.vue';
import MenuPanel from './components/MenuPanel.vue';
import { type } from 'os'

export default {
    mixins: [
        functionalityStyle,
        panelControl,
        contentControl,
    ],
    components: {
        RightArrowIcon,
        LeftArrowIcon,
        MenuBurger,
        MenuShadow,
        MenuPanel,
    },
    props: {
        source: {
            type: Object,
            default: (() => {
                return {
                    title: 'Default',
                    children: [
                        {
                            title: 'Default child 1',
                            link: '/browse/test',
                            children: []
                        }
                    ]
                }
            })
        },
        user: {
            type: Object,
            default: (() => {
                return {
                    loggedIn: false,
                    avatar: null,
                    username: 'Jhon Doe',
                    role: 'Seller',
                    logoutUrl: '/logout',
                    infoUrl: '/profile'
                }
            })
        },
        menuActivator: {
            type: Boolean,
            default: false
        },
        panelWidth: {
            type: Number,
            default: 300,
        },
        menuOpenSpeed: {
            type: Number,
            default: 350,
        },
        menuSwitchSpeed: {
            type: Number,
            default: 300,
        },
    },
    data() {
        return {
            data: this.source,
            isActive: this.menuActivator,
            isTranslating: false,
        };
    },
    mounted() {
        this.content_currentItem = this.data;
    },
    computed: {
        currentItemHasParent() {
            return this.content_parentStack.length >= 1;
        },
        prevItemHasParent() {
            return this.content_parentStack.length >= 2;
        },
    },
    methods: {
    	clickBurger() {
            this.isActive = !this.isActive;
            this.$emit('update:menuActivator', this.isActive);
    	},
        clickShadow() {
            this.isActive = false;
            this.$emit('update:menuActivator', false);
        },
        clickNextItem(targetItem) {

            if (this.isTranslating) {
                return;
            }

            // go to link
            if (targetItem.children.length <= 0) {
                // ...
                return;
            }

            this.slideToNext(targetItem);
        },
        clickPrevItem() {

            if (this.isTranslating || !this.currentItemHasParent) {
                return;
            }

            this.slideToPrev();
        },

        /*
         * main part of core
         * definite of how to handle panel sliding and item updating
         */
        slideToNext(targetItem) {

            // set target item as content of next panel
            this.content_setNextItem(targetItem);

            // switch animation on
            this.setTranslating(true);

            // activate panel sliding with animation after `.translating` class has updated to panel dom.
            this.$nextTick(() => {
                this.panel_slideNext();
            });

            // reset panel position
            this.homingAfterTranslatingNext();
        },
        slideToPrev() {

            // set prev item which is the parent of the current item.
            this.content_setPrevItem();

            // switch animation on
            this.setTranslating(true);

            // activate panel sliding with animation after `.translating` class has updated to panel dom.
            this.$nextTick(() => {
                this.panel_slideBack();
            });

            // reset panel position
            this.homingAfterTranslatingBack();
        },

        // handle homing after slide animation end
        homingAfterTranslatingNext() {
            setTimeout(() => {

                // switch off transition state of panel
                this.setTranslating(false);

                // push current to parent stack
                this.content_pushCurrentToParentStack();

                // homing
                this.panel_homingPosition(); // reset panel position just like the beginning.
                this.content_homingItemAfterNext(); // change item between these panels to meet updated panel position.
            }, this.menuSwitchSpeed);
        },
        homingAfterTranslatingBack() {
            setTimeout(() => {
                this.setTranslating(false);

                // homing
                this.panel_homingPosition();
                this.content_homingItemAfterBack();
            }, this.menuSwitchSpeed);
        },

        // utils
        setTranslating(status) {
            this.isTranslating = status;
        },
    },
};
</script>

<style scoped>
    .btn {
        background-color:transparent;
        border: 0.5px solid #33333350;
        padding:3px 8px 3px 8px;
        text-decoration:none;
        border-radius:2px;
        margin-right: 10px;
    }
    .btn-danger{
        color:#b91400;
    }
    .btn-primary{
        color:#009688;
    }
</style>
