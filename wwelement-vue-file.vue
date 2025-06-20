<template>
  <div class="feature-cards-section">
    <div class="container">
      <h2 v-if="content.sectionTitle" class="section-title">{{ content.sectionTitle }}</h2>
      <p v-if="content.sectionSubtitle" class="section-subtitle">{{ content.sectionSubtitle }}</p>
      
      <div class="features-grid" :style="gridStyle">
        <div 
          v-for="(feature, index) in content.features" 
          :key="index"
          class="feature"
          :class="{ 
            'hovered': hoveredIndex === index && content.enableHover,
            'featured': feature.featured
          }"
          @mouseenter="hoveredIndex = index"
          @mouseleave="hoveredIndex = null"
          @click="handleFeatureClick(feature, index)"
        >
          <div 
            class="feature-icon" 
            :style="iconStyle(index)"
          >
            <span v-if="feature.emoji" class="feature-emoji">{{ feature.emoji }}</span>
            <i v-else-if="feature.iconClass" :class="feature.iconClass"></i>
            <div v-else class="icon-placeholder"></div>
          </div>
          
          <h3 class="feature-title" :style="titleStyle">{{ feature.title }}</h3>
          <p class="feature-description" :style="descriptionStyle">{{ feature.description }}</p>
          
          <a 
            v-if="feature.ctaText && content.showCta"
            :href="feature.ctaLink || '#'"
            class="feature-cta"
            :style="ctaStyle"
            @click.stop="handleCtaClick($event, feature)"
          >
            {{ feature.ctaText }}
            <span class="arrow">→</span>
          </a>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'FeatureCards',
  props: {
    content: { type: Object, required: true }
  },
  emits: ['trigger-event'],
  data() {
    return {
      hoveredIndex: null
    }
  },
  computed: {
    gridStyle() {
      return {
        gridTemplateColumns: `repeat(auto-fit, minmax(${this.getMinWidth()}px, 1fr))`,
        gap: `${this.content.gap}px`
      }
    },
    titleStyle() {
      return {
        fontSize: `${this.content.titleSize}px`,
        fontWeight: this.content.titleWeight,
        color: this.content.titleColor
      }
    },
    descriptionStyle() {
      return {
        fontSize: `${this.content.descriptionSize}px`,
        color: this.content.descriptionColor
      }
    },
    ctaStyle() {
      return {
        color: this.content.accentColor
      }
    }
  },
  methods: {
    getMinWidth() {
      const columns = this.content.columns || 3;
      const widthMap = {
        1: 500,
        2: 400,
        3: 300,
        4: 250
      };
      return widthMap[columns] || 300;
    },
    iconStyle(index) {
      const isHovered = this.hoveredIndex === index && this.content.enableHover;
      return {
        backgroundColor: isHovered ? this.content.accentColorHover : this.content.accentColor,
        width: `${this.content.iconSize}px`,
        height: `${this.content.iconSize}px`,
        transform: isHovered ? 'scale(1.1) rotate(5deg)' : 'scale(1)'
      }
    },
    handleFeatureClick(feature, index) {
      this.$emit('trigger-event', {
        name: 'feature:click',
        event: { feature, index }
      });
    },
    handleCtaClick(event, feature) {
      if (feature.ctaLink && feature.ctaLink.startsWith('#')) {
        event.preventDefault();
      }
      this.$emit('trigger-event', {
        name: 'feature:cta-click',
        event: { feature }
      });
    }
  }
}
</script>

<style lang="scss" scoped>
.feature-cards-section {
  width: 100%;
  background-color: v-bind('content.backgroundColor');
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 80px 20px;
}

.section-title {
  text-align: center;
  margin: 0 0 20px 0;
  font-size: 36px;
  font-weight: 700;
  color: v-bind('content.titleColor');
}

.section-subtitle {
  text-align: center;
  font-size: 18px;
  color: v-bind('content.descriptionColor');
  margin: 0 auto 60px;
  max-width: 600px;
  line-height: 1.6;
}

.features-grid {
  display: grid;
}

.feature {
  text-align: center;
  padding: v-bind('content.cardPadding + "px"');
  background: v-bind('content.cardBackground');
  border-radius: v-bind('content.borderRadius + "px"');
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  cursor: pointer;
  position: relative;
  overflow: hidden;

  &::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 3px;
    background: v-bind('content.accentColor');
    transform: translateX(-100%);
    transition: transform 0.3s ease;
  }

  &.hovered {
    transform: translateY(-5px);
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);

    &::before {
      transform: translateX(0);
    }

    .feature-cta {
      opacity: 1;
      transform: translateY(0);
    }
  }

  &.featured {
    border: 2px solid v-bind('content.accentColor');
  }
}

.feature-icon {
  border-radius: 50%;
  margin: 0 auto 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
}

.feature-emoji {
  font-size: calc(v-bind('content.iconSize + "px"') * 0.5);
  line-height: 1;
}

.icon-placeholder {
  width: 50%;
  height: 50%;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 4px;
}

.feature-title {
  margin: 0 0 15px 0;
  transition: color 0.3s ease;
}

.feature.hovered .feature-title {
  color: v-bind('content.enableHover ? content.accentColor : content.titleColor');
}

.feature-description {
  margin: 0;
  line-height: 1.6;
}

.feature-cta {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  text-decoration: none;
  font-weight: 500;
  margin-top: 15px;
  opacity: 0;
  transform: translateY(10px);
  transition: all 0.3s ease;

  &:hover {
    gap: 10px;

    .arrow {
      transform: translateX(3px);
    }
  }
}

.arrow {
  transition: transform 0.3s ease;
  display: inline-block;
}

@media (max-width: 768px) {
  .container {
    padding: 60px 20px;
  }
  
  .section-title {
    font-size: 28px;
  }
  
  .section-subtitle {
    font-size: 16px;
    margin-bottom: 40px;
  }
  
  .feature {
    padding: 30px;
  }
}
</style>