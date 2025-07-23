# ELITS Helper

**Version:** v0.02  
**Creator:** lpt2007

> [!IMPORTANT] 
> **Supported cards:**
>  - Button
>  - Calendar
>  - Climate
>  - Cover
>  - Horizontal Buttons Stack
>  - Media Player
>  - Pop Up
>  - Select
>  - Separator

Advanced helper module with conditional logic.

Features:
- Entity badges: Display entity states or attributes on sub-buttons
- Icon badges: Show icon indicators on sub-buttons
- Sub-button backgrounds: Change background colors based on conditions
- Icon effects: Apply animations to main and sub-button icons
- Badge effects: Apply animations to badges
- Main icon background: Change the main icon background conditionally
- Sub-button effects: Apply animations (including color for glow) to the entire sub-button container

All features support Home Assistant-style conditions for state-based logic.

Configure this module via the editor or in YAML, for example:

```yaml
elits-helper: 
    entity_badges: YOUR_VALUE
```

---

<details>

<summary><b>🧩 Get this Module</b></summary>

<br>

> To use this module, simply install it from the Module Store (from the editor of any card > Modules), or copy and paste the following configuration into your `/www/bubble/bubble-modules.yaml` file.

```yaml
elits-helper:
    name: "ELITS Helper"
    version: "v0.02"
    creator: "lpt2007"
    link: "https://github.com/Clooos/Bubble-Card/discussions/XXXX"

    supported:
        - button
        - calendar
        - climate
        - cover
        - horizontal-buttons-stack
        - media-player
        - pop-up
        - select
        - separator

    description: |
        Advanced helper module with conditional logic.
        
        Features:
        - Entity badges: Display entity states or attributes on sub-buttons
        - Icon badges: Show icon indicators on sub-buttons
        - Sub-button backgrounds: Change background colors based on conditions
        - Icon effects: Apply animations to main and sub-button icons
        - Badge effects: Apply animations to badges
        - Main icon background: Change the main icon background conditionally
        - Sub-button effects: Apply animations (including color for glow) to the entire sub-button container
        
        All features support Home Assistant-style conditions for state-based logic.
        
        <br><br>
        <code-block><pre>
        elits-helper: 
            entity_badges: YOUR_VALUE
        </pre></code-block>

    code: |
        /* =============================
           CSS for the badge visuals
           ============================= */
        .bubble-sub-button-badge {
          position: absolute;
          min-width: 18px;
          height: 16px;
          border-radius: 8px;
          background-color: var(--badge-color, var(--primary-color));
          color: white;
          font-size: 11px;
          font-weight: bold;
          display: flex;
          align-items: center;
          justify-content: center;
          padding: 0 4px;
          z-index: 1; /* reduziert, damit Badges nicht Popups überlagern */
          box-sizing: border-box;
          box-shadow: 0 1px 2px rgba(0,0,0,0.2);
          transform: scale(var(--badge-scale, 1));
          transform-origin: center;
          pointer-events: none;
        }
        .entity-badge { top: 0; right: 0; }
        .icon-badge   { top: 0; left: 0;  }
        .without-background {
          background-color: transparent;
          box-shadow: none;
        }
        .badge-icon {
          display: flex;
          align-items: center;
          justify-content: center;
          width: 13px;
          height: 13px;
        }
        
        /* =============================
           Direct styling for sub-button backgrounds
           ============================= */
        .sub-button-direct-bg {
          /* Used to track that we are controlling the background color with JS */
        }
        
        /* =============================
           Animations for effects
           ============================= */
        @keyframes pulse-scale {
          0%, 100% { transform: scale(1); }
          50%      { transform: scale(1.05); }
        }
        @keyframes gentle-rotate {
          0% { transform: rotate(0deg); }
          25% { transform: rotate(5deg); }
          75% { transform: rotate(-5deg); }
          100% { transform: rotate(0deg); }
        }
        @keyframes border-pulse {
          0%, 100% { box-shadow: 0 0 0 transparent; }
          50%      { box-shadow: 0 0 10px var(--effect-color, rgba(255,255,255,0.7)); }
        }
        @keyframes gentle-shake {
          0%, 100% { transform: translateX(0); }
          25%      { transform: translateX(-2px); }
          75%      { transform: translateX(2px); }
        }
        @keyframes float-up-down {
          0%, 100% { transform: translateY(0); }
          50%      { transform: translateY(-3px); }
        }
        @keyframes zoom {
          0%   { transform: scale(1); }
          50%  { transform: scale(1.15); }
          100% { transform: scale(1); }
        }
        @keyframes heartbeat {
          0%   { transform: scale(1); }
          25%  { transform: scale(1.07); }
          40%  { transform: scale(1); }
          60%  { transform: scale(1.07); }
          100% { transform: scale(1); }
        }
        
        /* New custom animations: spin-360, flip, vibrate */
        @keyframes spin360 {
          0% { transform: rotate(0deg); }
          100% { transform: rotate(360deg); }
        }
        @keyframes flip {
          0% { transform: rotateY(0deg); }
          50% { transform: rotateY(180deg); }
          100% { transform: rotateY(360deg); }
        }
        @keyframes vibrate {
          0%   { transform: translate(0,0); }
          20%  { transform: translate(-1px,0); }
          40%  { transform: translate(1px,0); }
          60%  { transform: translate(-1px,0); }
          80%  { transform: translate(1px,0); }
          100% { transform: translate(0,0); }
        }
        
        .effect-pulse       { animation: pulse-scale 2s infinite ease-in-out; }
        .effect-rotate      { animation: gentle-rotate 3s infinite ease-in-out; }
        .effect-glow        { animation: border-pulse 2s infinite ease-in-out; }
        .effect-shake       { animation: gentle-shake 0.8s infinite ease-in-out; }
        .effect-float       { animation: float-up-down 3s infinite ease-in-out; }
        .effect-zoom        { animation: zoom 1.5s infinite ease-in-out; }
        .effect-heartbeat   { animation: heartbeat 2s infinite ease-in-out; }
        
        .effect-spin-360    { animation: spin360 2s linear infinite; }
        .effect-flip        { animation: flip 2s linear infinite; }
        .effect-vibrate     { animation: vibrate 0.6s linear infinite; }
        
        /* For sub-button icon only */
        .bubble-sub-button-icon.has-effect {
          position: relative;
          transform-origin: center;
          display: inline-flex;
        }
        /* For main icon only */
        .bubble-icon.has-effect {
          position: relative;
          transform-origin: center;
          display: inline-flex;
        }
        
        ${(() => {
          try {
            // ===== Global variables for tracking =====
            const DEBUG = false;
            let trackedEntities = new Set();
        
            // ===== Helper functions for conditions =====
            function evaluateSingleCondition(condObj, hass) {
              if (!condObj || typeof condObj !== 'object') return false;
              if (!condObj.condition) return false;
              const t = condObj.condition;
              switch (t) {
                case 'state': {
                  const eid = condObj.entity_id || condObj.entity;
                  if (!eid || !hass.states[eid]) return false;
                  let val;
                  if (condObj.attribute) {
                    val = hass.states[eid].attributes[condObj.attribute];
                    if (val === undefined) return false;
                  } else {
                    val = hass.states[eid].state;
                  }
                  if (Array.isArray(condObj.state)) {
                    return condObj.state.includes(val);
                  } else {
                    return val === condObj.state;
                  }
                }
                case 'numeric_state': {
                  const eid = condObj.entity_id || condObj.entity;
                  if (!eid || !hass.states[eid]) return false;
                  let val;
                  if (condObj.attribute) {
                    val = hass.states[eid].attributes[condObj.attribute];
                  } else {
                    val = hass.states[eid].state;
                  }
                  const numVal = parseFloat(val);
                  if (isNaN(numVal)) return false;
                  let ok = true;
                  if (condObj.above !== undefined) {
                    ok = ok && (numVal > parseFloat(condObj.above));
                  }
                  if (condObj.below !== undefined) {
                    ok = ok && (numVal < parseFloat(condObj.below));
                  }
                  return ok;
                }
                case 'and':
                  if (!Array.isArray(condObj.conditions)) return false;
                  return condObj.conditions.every(sc => evaluateSingleCondition(sc, hass));
                case 'or':
                  if (!Array.isArray(condObj.conditions)) return false;
                  return condObj.conditions.some(sc => evaluateSingleCondition(sc, hass));
                case 'not':
                  if (Array.isArray(condObj.conditions) && condObj.conditions.length > 0) {
                    return !evaluateSingleCondition(condObj.conditions[0], hass);
                  }
                  if (condObj.condition_obj) {
                    return !evaluateSingleCondition(condObj.condition_obj, hass);
                  }
                  return false;
                default:
                  return false;
              }
            }
        
            function checkAllConditions(cProp, hass) {
              if (!cProp) return true;
              if (Array.isArray(cProp)) {
                return cProp.every(x => evaluateSingleCondition(x, hass));
              }
              return evaluateSingleCondition(cProp, hass);
            }
        
            function processColor(color) {
              if (!color) return null;
              if (color.startsWith('#') || color.startsWith('rgb') || color.startsWith('hsl')) {
                return color;
              }
              return `var(--${color}-color)`;
            }
        
            // ===== Find entities in conditions =====
            function collectEntitiesFromCondition(condObj) {
              if (!condObj || typeof condObj !== 'object') return [];
              const entities = [];
              const t = condObj.condition;
              if (!t) return entities;
              switch (t) {
                case 'state':
                case 'numeric_state': {
                  const eid = condObj.entity_id || condObj.entity;
                  if (eid && typeof eid === 'string') entities.push(eid);
                  break;
                }
                case 'and':
                case 'or':
                  if (Array.isArray(condObj.conditions)) {
                    condObj.conditions.forEach(sc => {
                      entities.push(...collectEntitiesFromCondition(sc));
                    });
                  }
                  break;
                case 'not':
                  if (Array.isArray(condObj.conditions) && condObj.conditions.length > 0) {
                    entities.push(...collectEntitiesFromCondition(condObj.conditions[0]));
                  }
                  if (condObj.condition_obj) {
                    entities.push(...collectEntitiesFromCondition(condObj.condition_obj));
                  }
                  break;
              }
              return entities;
            }
        
            // ===== Collect all relevant entities =====
            function collectAllTrackedEntities() {
              const cfg = this.config["elits-helper"] || {};
              const allEntities = new Set();
        
              // 1) Entity-Badges
              const eBadges = cfg.entity_badges || {};
              for (const bc of Object.values(eBadges)) {
                if (bc?.entity) allEntities.add(bc.entity);
                if (bc?.condition) {
                  collectEntitiesFromCondition(bc.condition).forEach(e => allEntities.add(e));
                }
              }
              // 2) Icon-Badges
              const iBadges = cfg.icon_badges || {};
              for (const bc of Object.values(iBadges)) {
                if (bc?.condition) {
                  collectEntitiesFromCondition(bc.condition).forEach(e => allEntities.add(e));
                }
              }
              // 3) Sub-Button Backgrounds
              const sbBg = cfg.sub_buttons_background || {};
              for (const bgC of Object.values(sbBg)) {
                if (bgC?.condition) {
                  collectEntitiesFromCondition(bgC.condition).forEach(e => allEntities.add(e));
                }
              }
              // 4) Icon Effects
              const iE = cfg.icon_effects || {};
              if (iE.main_icon?.condition) {
                collectEntitiesFromCondition(iE.main_icon.condition).forEach(e => allEntities.add(e));
              }
              const sbIcons = iE.sub_button_icon || {};
              for (const eC of Object.values(sbIcons)) {
                if (eC?.condition) {
                  collectEntitiesFromCondition(eC.condition).forEach(e => allEntities.add(e));
                }
              }
              // 5) Entity Badge Effects
              const eBE = cfg.entity_badge_effects || {};
              for (const ef of Object.values(eBE)) {
                if (ef?.condition) {
                  collectEntitiesFromCondition(ef.condition).forEach(e => allEntities.add(e));
                }
              }
              // 6) Icon Badge Effects
              const iBE = cfg.icon_badge_effects || {};
              for (const ef of Object.values(iBE)) {
                if (ef?.condition) {
                  collectEntitiesFromCondition(ef.condition).forEach(e => allEntities.add(e));
                }
              }
              // 7) Main Icon Background
              const mBg = cfg.main_icon_background || {};
              if (mBg?.condition) {
                collectEntitiesFromCondition(mBg.condition).forEach(e => allEntities.add(e));
              }
              // 8) Sub-Button Effects
              const sBE = cfg.sub_button_effects || {};
              for (const eff of Object.values(sBE)) {
                if (eff?.condition) {
                  collectEntitiesFromCondition(eff.condition).forEach(e => allEntities.add(e));
                }
              }
              return allEntities;
            }
        
            // ===== Set up state change listener =====
            function setupStateChangeListener() {
              const cardEl = card;
              const connection = hass.connection;
              if (cardEl._badgyStateChangeHandler) {
                connection.removeEventListener('state_changed', cardEl._badgyStateChangeHandler);
              }
              cardEl._badgyStateChangeHandler = (event) => {
                const entityId = event.data.entity_id;
                if (trackedEntities.has(entityId)) {
                  if (DEBUG) console.log(`[badgy-helper] State changed for tracked entity: ${entityId}`);
                  processEntityBadges();
                  processIconBadges();
                  processSubButtonBackgrounds();
                  processIconEffects();
                  processSubButtonEffects();
                  processEntityBadgeEffects();
                  processIconBadgeEffects();
                  processMainIconBackground();
                }
              };
              connection.addEventListener('state_changed', cardEl._badgyStateChangeHandler);
              if (DEBUG) console.log('[badgy-helper] Registered state_changed listener');
            }
        
            // ===== Clean up event listeners when card is disconnected =====
            if (!card._badgyCleanupHandlerAdded) {
              card._badgyCleanupHandlerAdded = true;
              const originalDisconnectedCallback = card.disconnectedCallback;
              card.disconnectedCallback = function() {
                if (DEBUG) console.log('[badgy-helper] Card removed, cleaning up...');
                if (this._badgyStateChangeHandler && hass.connection) {
                  hass.connection.removeEventListener('state_changed', this._badgyStateChangeHandler);
                  this._badgyStateChangeHandler = null;
                }
                if (originalDisconnectedCallback) {
                  originalDisconnectedCallback.call(this);
                }
              };
            }
        
            // ===== Read configuration =====
            const config = this.config["elits-helper"] || {};
            const cardEl = card;
        
            // ========== PROCESS: Entity Badges ==========
            // ========== PROCESS: Entity Badges ==========
            function processEntityBadges() {
              const allEntityBadgeGroups = [
                config.entity_badges || {},
                config.entity_badges_el1 || {},
                config.entity_badges_el2 || {}, // <-- Dodaj več po potrebi
              ];
              const currentEntityBadges = new Set();
        
              for (const eBadges of allEntityBadgeGroups) {
                for (const [k, bc] of Object.entries(eBadges)) {
                  if (!bc?.entity) continue;
                  const sbNum = parseInt(k) + 1;
                  const stObj = hass.states[bc.entity];
                  if (!stObj) continue;
        
                  const conditionMet = checkAllConditions(bc.condition, hass);
                  const showBadge = conditionMet && !(bc.show_if_off === false && stObj.state === 'off');
                  if (!showBadge) continue;
        
                  let dispVal;
                  if (bc.attribute && stObj.attributes[bc.attribute] !== undefined) {
                    dispVal = stObj.attributes[bc.attribute];
                  } else {
                    dispVal = stObj.state;
                  }
        
                  if (bc.decimals !== undefined && !isNaN(parseFloat(dispVal))) {
                    dispVal = parseFloat(dispVal).toFixed(parseInt(bc.decimals));
                  }
        
                  if (bc.show_unit !== false && !bc.attribute && stObj.attributes.unit_of_measurement) {
                    dispVal += stObj.attributes.unit_of_measurement;
                  }
        
                  currentEntityBadges.add(`${sbNum}`);
        
                  createBadge({
                    subButtonNumber: sbNum,
                    content: dispVal,
                    badgeColor: bc.color,
                    textColor: bc.text_color,
                    hPos: bc.h_pos ?? -8,
                    vPos: bc.v_pos ?? -8,
                    scale: bc.scale ?? 1,
                    showBackground: bc.show_background !== false,
                    badgeType: 'entity-badge'
                  });
                }
              }
        
              // Odstrani samo tiste, ki jih nismo prikazali
              for (let i = 1; i <= 10; i++) {
                if (!currentEntityBadges.has(`${i}`)) {
                  const sb = cardEl.querySelector(`.bubble-sub-button-${i}`);
                  if (sb) {
                    const badge = sb.querySelector('.entity-badge');
                    if (badge) sb.removeChild(badge);
                  }
                }
              }
            }
            // ========== PROCESS: Icon Badges ==========
            function processIconBadges(){
              const allIconBadgeGroups = [
                config.icon_badges || {},
                config.icon_badges_el1 || {},
                config.icon_badges_el2 || {},  // ✅ DODANO
              ];
              const currentIconBadges = new Set();
        
              for (const iBadges of allIconBadgeGroups) {
                for (const [k, bc] of Object.entries(iBadges)) {
                  if (!bc?.icon) continue;
                  const sbNum = parseInt(k) + 1;
                  const conditionMet = checkAllConditions(bc.condition, hass);
                  if (conditionMet) {
                    currentIconBadges.add(`${sbNum}-${bc.icon}`);  // Uporabimo kombinacijo, da ločimo
                    createBadge({
                      subButtonNumber: sbNum,
                      icon: bc.icon,
                      badgeColor: bc.color,
                      textColor: bc.text_color,
                      hPos: bc.h_pos ?? -8,
                      vPos: bc.v_pos ?? -8,
                      scale: bc.scale ?? 1,
                      showBackground: bc.show_background !== false,
                      badgeType: 'icon-badge'
                    });
                  }
                }
              }
        
              // Odstrani samo tiste, ki jih nismo pokazali
              for (let i = 1; i <= 10; i++) {
                const sb = cardEl.querySelector(`.bubble-sub-button-${i}`);
                if (!sb) continue;
                const badges = sb.querySelectorAll('.icon-badge');
                for (const badge of badges) {
                  const currentIcon = badge.querySelector('ha-icon')?.getAttribute('icon') || '';
                  const key = `${i}-${currentIcon}`;
                  if (!currentIconBadges.has(key)) {
                    sb.removeChild(badge);
                  }
                }
              }
            }
        
        
            // ========== HELPER: createBadge ==========
            function createBadge({subButtonNumber, content, icon, badgeColor, textColor, hPos, vPos, scale, showBackground, badgeType}) {
              const sb = cardEl.querySelector(`.bubble-sub-button-${subButtonNumber}`);
              if (!sb) return;
              if (getComputedStyle(sb).position === 'static') {
                sb.style.position = 'relative';
              }
              const old = sb.querySelector(`.${badgeType}`);
              if (old) sb.removeChild(old);
              const badge = document.createElement('div');
              badge.className = `bubble-sub-button-badge ${badgeType} ` + (showBackground ? '' : 'without-background');
              if (scale !== 1) {
                badge.style.setProperty('--badge-scale', scale);
              }
              if (badgeType === 'entity-badge') {
                badge.style.top = `${vPos}px`;
                badge.style.right = `${hPos}px`;
              } else {
                badge.style.top = `${vPos}px`;
                badge.style.left = `${hPos}px`;
              }
              badge.dataset.hPos = hPos;
              badge.dataset.vPos = vPos;
              if (badgeColor) badge.style.setProperty('--badge-color', processColor(badgeColor));
              if (textColor) badge.style.color = processColor(textColor);
              if (icon) {
                const ic = document.createElement('ha-icon');
                ic.setAttribute('icon', icon);
                ic.className = 'badge-icon';
                badge.appendChild(ic);
              } else if (content) {
                badge.textContent = content;
              }
              sb.appendChild(badge);
            }
        
            // ========== PROCESS: Sub-Button Backgrounds ==========
            function processSubButtonBackgrounds(){
              const sbBg = config.sub_buttons_background || {};
              for (let i = 1; i <= 10; i++) {
                const sb = cardEl.querySelector(`.bubble-sub-button-${i}`);
                if (!sb) continue;
                sb.classList.remove('sub-button-direct-bg');
                sb.style.removeProperty('background');
                sb.style.removeProperty('opacity');
              }
              for (const [k, bgC] of Object.entries(sbBg)) {
                if (!bgC) continue;
                const sbN = parseInt(k) + 1;
                const conditionMet = checkAllConditions(bgC.condition, hass);
                if (conditionMet) {
                  const type = bgC.type ?? 'solid';
                  const op = (bgC.opacity !== undefined) ? bgC.opacity : 1.0;
                  let colorStr = '';
                  if (type === 'solid') {
                    if (bgC.color) {
                      colorStr = processColor(bgC.color);
                    } else {
                      continue;
                    }
                  }
                  else if (type === 'two-color') {
                    const c1 = processColor(bgC.color1 || '#ff0000');
                    const c2 = processColor(bgC.color2 || '#0000ff');
                    colorStr = `linear-gradient(135deg, ${c1} 0%, ${c2} 100%)`;
                  }
                  else if (type === 'three-color') {
                    const c1 = processColor(bgC.color1 || '#ff0000');
                    const c2 = processColor(bgC.color2 || '#00ff00');
                    const c3 = processColor(bgC.color3 || '#0000ff');
                    colorStr = `linear-gradient(135deg, ${c1} 0%, ${c2} 50%, ${c3} 100%)`;
                  }
                  applySubButtonBg(sbN, colorStr, op);
                }
              }
            }
        
            function applySubButtonBg(subButtonNum, bgValue, opacity) {
              const sb = cardEl.querySelector(`.bubble-sub-button-${subButtonNum}`);
              if (!sb) return;
              sb.classList.add('sub-button-direct-bg');
              sb.style.setProperty('background', bgValue, 'important');
              sb.style.setProperty('opacity', opacity, 'important');
            }
        
            // ========== PROCESS: Icon Effects (for Main Icon and Sub-Button Icons) ==========
            function processIconEffects(){
              const iE = config.icon_effects || {};
              const mCfg = iE.main_icon || {};
              const mainIconEl = cardEl.querySelector('.bubble-icon');
              if (mainIconEl) {
                mainIconEl.classList.remove(
                  'effect-pulse','effect-rotate','effect-glow','effect-shake',
                  'effect-float','effect-zoom','effect-heartbeat','effect-spin-360',
                  'effect-flip','effect-vibrate','has-effect'
                );
                mainIconEl.style.removeProperty('--effect-color');
                const conditionMet = checkAllConditions(mCfg.condition, hass);
                if (conditionMet && mCfg.effect) {
                  mainIconEl.classList.add(`effect-${mCfg.effect}`, 'has-effect');
                  if (mCfg.color) {
                    mainIconEl.style.setProperty('--effect-color', processColor(mCfg.color));
                  }
                }
              }
              // Neu: Sub-Button ICON Effekte – über Schleife von 0 bis 9
              for (let i = 0; i < 10; i++) {
                const eC = iE.sub_button_icon && iE.sub_button_icon[i.toString()];
                if (eC && eC.effect) {
                  const conditionMet = checkAllConditions(eC.condition, hass);
                  if (conditionMet) {
                    const el = cardEl.querySelector(`.bubble-sub-button-${i+1} .bubble-sub-button-icon`);
                    if (el) {
                      el.classList.add(`effect-${eC.effect}`, 'has-effect');
                      if (eC.color) {
                        el.style.setProperty('--effect-color', processColor(eC.color));
                      }
                    }
                  }
                }
              }
            }
        
            // ========== PROCESS: Sub-Button Effects (on the container) ==========
            function processSubButtonEffects(){
              const sBE = config.sub_button_effects || {};
              for (let i = 0; i < 10; i++){
                const eff = sBE && sBE[i.toString()];
                if (eff && eff.effect) {
                  const conditionMet = checkAllConditions(eff.condition, hass);
                  if (conditionMet) {
                    const sb = cardEl.querySelector(`.bubble-sub-button-${i+1}`);
                    if (sb) {
                      sb.classList.add(`effect-${eff.effect}`, 'has-effect');
                      if (eff.color) {
                        sb.style.setProperty('--effect-color', processColor(eff.color));
                      }
                    }
                  }
                }
              }
            }
        
            // ========== PROCESS: Entity-Badge Effects ==========
            function processEntityBadgeEffects(){
              const eBE = config.entity_badge_effects || {};
              const allEntBadges = cardEl.querySelectorAll('.bubble-sub-button-badge.entity-badge');
              allEntBadges.forEach(b => {
                b.classList.remove(
                  'effect-pulse','effect-rotate','effect-glow','effect-shake',
                  'effect-float','effect-zoom','effect-heartbeat','effect-spin-360',
                  'effect-flip','effect-vibrate','has-effect'
                );
                b.style.removeProperty('--effect-color');
              });
              for (const [k, ef] of Object.entries(eBE)){
                if (!ef?.effect) continue;
                const sbN = parseInt(k) + 1;
                const conditionMet = checkAllConditions(ef.condition, hass);
                if (conditionMet) {
                  const badge = cardEl.querySelector(`.bubble-sub-button-${sbN} .entity-badge`);
                  if (!badge) continue;
                  badge.classList.add(`effect-${ef.effect}`, 'has-effect');
                  if (ef.color) {
                    badge.style.setProperty('--effect-color', processColor(ef.color));
                  }
                }
              }
            }
        
            // ========== PROCESS: Icon-Badge Effects ==========
            function processIconBadgeEffects(){
              const iBE = config.icon_badge_effects || {};
              const allIconBadges = cardEl.querySelectorAll('.bubble-sub-button-badge.icon-badge');
              allIconBadges.forEach(b => {
                b.classList.remove(
                  'effect-pulse','effect-rotate','effect-glow','effect-shake',
                  'effect-float','effect-zoom','effect-heartbeat','effect-spin-360',
                  'effect-flip','effect-vibrate','has-effect'
                );
                b.style.removeProperty('--effect-color');
              });
              for (const [k, ef] of Object.entries(iBE)){
                if (!ef?.effect) continue;
                const sbN = parseInt(k) + 1;
                const conditionMet = checkAllConditions(ef.condition, hass);
                if (conditionMet) {
                  const badge = cardEl.querySelector(`.bubble-sub-button-${sbN} .icon-badge`);
                  if (!badge) continue;
                  badge.classList.add(`effect-${ef.effect}`, 'has-effect');
                  if (ef.color) {
                    badge.style.setProperty('--effect-color', processColor(ef.color));
                  }
                }
              }
            }
        
            // ========== PROCESS: Main Icon Background ==========
            function processMainIconBackground(){
              const mBg = config.main_icon_background || {};
              const mainCont = cardEl.querySelector('.bubble-icon-container');
              if (!mainCont) return;
              mainCont.style.removeProperty('background');
              mainCont.style.removeProperty('opacity');
              const conditionMet = checkAllConditions(mBg.condition, hass);
              if (conditionMet && mBg.color) {
                const op = (mBg.opacity !== undefined) ? mBg.opacity : 1.0;
                const cProc = processColor(mBg.color);
                mainCont.style.setProperty('background', cProc, 'important');
                mainCont.style.setProperty('opacity', op, 'important');
              }
            }
        
            // ===== Start: State tracking, Listener, and Initial Pass =====
            trackedEntities.clear();
            collectAllTrackedEntities.call(this).forEach(e => trackedEntities.add(e));
            if (DEBUG) console.log('[badgy-helper] Entities tracked:', [...trackedEntities]);
            setupStateChangeListener.call(this);
        
            processEntityBadges();
            processIconBadges();
            processSubButtonBackgrounds();
            processIconEffects();
            processSubButtonEffects();
            processEntityBadgeEffects();
            processIconBadgeEffects();
            processMainIconBackground();
        
          } catch (err) {
            console.error("Error in elits-helper module:", err);
          }
          return '';
        })()}
        

    editor:
      - type: expandable
        title: Entity Badges
        icon: mdi:tag-text
        name: entity_badges
        schema:
          - type: expandable
            title: Sub-Button 1
            name: '0'
            schema:
              - name: entity
                label: Entity
                selector:
                  entity: {}
              - name: attribute
                label: Attribute (optional)
                selector:
                  attribute: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: decimals
                label: Decimal Places (for numbers)
                selector:
                  number:
                    min: 0
                    max: 5
                    mode: slider
                    step: 1
              - name: show_unit
                label: Show Unit
                default: true
                selector:
                  boolean: {}
              - name: show_if_off
                label: Show When 'off'
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 2
            name: '1'
            schema:
              - name: entity
                label: Entity
                selector:
                  entity: {}
              - name: attribute
                label: Attribute (optional)
                selector:
                  attribute: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: decimals
                label: Decimal Places (for numbers)
                selector:
                  number:
                    min: 0
                    max: 5
                    mode: slider
                    step: 1
              - name: show_unit
                label: Show Unit
                default: true
                selector:
                  boolean: {}
              - name: show_if_off
                label: Show When 'off'
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 3
            name: '2'
            schema:
              - name: entity
                label: Entity
                selector:
                  entity: {}
              - name: attribute
                label: Attribute (optional)
                selector:
                  attribute: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: decimals
                label: Decimal Places (for numbers)
                selector:
                  number:
                    min: 0
                    max: 5
                    mode: slider
                    step: 1
              - name: show_unit
                label: Show Unit
                default: true
                selector:
                  boolean: {}
              - name: show_if_off
                label: Show When 'off'
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 4
            name: '3'
            schema:
              - name: entity
                label: Entity
                selector:
                  entity: {}
              - name: attribute
                label: Attribute (optional)
                selector:
                  attribute: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: decimals
                label: Decimal Places (for numbers)
                selector:
                  number:
                    min: 0
                    max: 5
                    mode: slider
                    step: 1
              - name: show_unit
                label: Show Unit
                default: true
                selector:
                  boolean: {}
              - name: show_if_off
                label: Show When 'off'
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 5
            name: '4'
            schema:
              - name: entity
                label: Entity
                selector:
                  entity: {}
              - name: attribute
                label: Attribute (optional)
                selector:
                  attribute: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: decimals
                label: Decimal Places (for numbers)
                selector:
                  number:
                    min: 0
                    max: 5
                    mode: slider
                    step: 1
              - name: show_unit
                label: Show Unit
                default: true
                selector:
                  boolean: {}
              - name: show_if_off
                label: Show When 'off'
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 6
            name: '5'
            schema:
              - name: entity
                label: Entity
                selector:
                  entity: {}
              - name: attribute
                label: Attribute (optional)
                selector:
                  attribute: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: decimals
                label: Decimal Places (for numbers)
                selector:
                  number:
                    min: 0
                    max: 5
                    mode: slider
                    step: 1
              - name: show_unit
                label: Show Unit
                default: true
                selector:
                  boolean: {}
              - name: show_if_off
                label: Show When 'off'
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
      - type: expandable
        title: Entity Badges EL1
        icon: mdi:tag-text
        name: entity_badges_el1
        schema:
          - type: expandable
            title: Sub-Button 1
            name: '0'
            schema:
              - name: entity
                label: Entity
                selector:
                  entity: {}
              - name: attribute
                label: Attribute (optional)
                selector:
                  attribute: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: decimals
                label: Decimal Places (for numbers)
                selector:
                  number:
                    min: 0
                    max: 5
                    mode: slider
                    step: 1
              - name: show_unit
                label: Show Unit
                default: true
                selector:
                  boolean: {}
              - name: show_if_off
                label: Show When 'off'
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 2
            name: '1'
            schema:
              - name: entity
                label: Entity
                selector:
                  entity: {}
              - name: attribute
                label: Attribute (optional)
                selector:
                  attribute: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: decimals
                label: Decimal Places (for numbers)
                selector:
                  number:
                    min: 0
                    max: 5
                    mode: slider
                    step: 1
              - name: show_unit
                label: Show Unit
                default: true
                selector:
                  boolean: {}
              - name: show_if_off
                label: Show When 'off'
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 3
            name: '2'
            schema:
              - name: entity
                label: Entity
                selector:
                  entity: {}
              - name: attribute
                label: Attribute (optional)
                selector:
                  attribute: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: decimals
                label: Decimal Places (for numbers)
                selector:
                  number:
                    min: 0
                    max: 5
                    mode: slider
                    step: 1
              - name: show_unit
                label: Show Unit
                default: true
                selector:
                  boolean: {}
              - name: show_if_off
                label: Show When 'off'
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 4
            name: '3'
            schema:
              - name: entity
                label: Entity
                selector:
                  entity: {}
              - name: attribute
                label: Attribute (optional)
                selector:
                  attribute: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: decimals
                label: Decimal Places (for numbers)
                selector:
                  number:
                    min: 0
                    max: 5
                    mode: slider
                    step: 1
              - name: show_unit
                label: Show Unit
                default: true
                selector:
                  boolean: {}
              - name: show_if_off
                label: Show When 'off'
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 5
            name: '4'
            schema:
              - name: entity
                label: Entity
                selector:
                  entity: {}
              - name: attribute
                label: Attribute (optional)
                selector:
                  attribute: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: decimals
                label: Decimal Places (for numbers)
                selector:
                  number:
                    min: 0
                    max: 5
                    mode: slider
                    step: 1
              - name: show_unit
                label: Show Unit
                default: true
                selector:
                  boolean: {}
              - name: show_if_off
                label: Show When 'off'
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 6
            name: '5'
            schema:
              - name: entity
                label: Entity
                selector:
                  entity: {}
              - name: attribute
                label: Attribute (optional)
                selector:
                  attribute: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: decimals
                label: Decimal Places (for numbers)
                selector:
                  number:
                    min: 0
                    max: 5
                    mode: slider
                    step: 1
              - name: show_unit
                label: Show Unit
                default: true
                selector:
                  boolean: {}
              - name: show_if_off
                label: Show When 'off'
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
      - type: expandable
        title: Entity Badges EL2
        icon: mdi:tag-text
        name: entity_badges_el2
        schema:
          - type: expandable
            title: Sub-Button 1
            name: '0'
            schema:
              - name: entity
                label: Entity
                selector:
                  entity: {}
              - name: attribute
                label: Attribute (optional)
                selector:
                  attribute: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: decimals
                label: Decimal Places (for numbers)
                selector:
                  number:
                    min: 0
                    max: 5
                    mode: slider
                    step: 1
              - name: show_unit
                label: Show Unit
                default: true
                selector:
                  boolean: {}
              - name: show_if_off
                label: Show When 'off'
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 2
            name: '1'
            schema:
              - name: entity
                label: Entity
                selector:
                  entity: {}
              - name: attribute
                label: Attribute (optional)
                selector:
                  attribute: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: decimals
                label: Decimal Places (for numbers)
                selector:
                  number:
                    min: 0
                    max: 5
                    mode: slider
                    step: 1
              - name: show_unit
                label: Show Unit
                default: true
                selector:
                  boolean: {}
              - name: show_if_off
                label: Show When 'off'
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 3
            name: '2'
            schema:
              - name: entity
                label: Entity
                selector:
                  entity: {}
              - name: attribute
                label: Attribute (optional)
                selector:
                  attribute: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: decimals
                label: Decimal Places (for numbers)
                selector:
                  number:
                    min: 0
                    max: 5
                    mode: slider
                    step: 1
              - name: show_unit
                label: Show Unit
                default: true
                selector:
                  boolean: {}
              - name: show_if_off
                label: Show When 'off'
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 4
            name: '3'
            schema:
              - name: entity
                label: Entity
                selector:
                  entity: {}
              - name: attribute
                label: Attribute (optional)
                selector:
                  attribute: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: decimals
                label: Decimal Places (for numbers)
                selector:
                  number:
                    min: 0
                    max: 5
                    mode: slider
                    step: 1
              - name: show_unit
                label: Show Unit
                default: true
                selector:
                  boolean: {}
              - name: show_if_off
                label: Show When 'off'
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 5
            name: '4'
            schema:
              - name: entity
                label: Entity
                selector:
                  entity: {}
              - name: attribute
                label: Attribute (optional)
                selector:
                  attribute: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: decimals
                label: Decimal Places (for numbers)
                selector:
                  number:
                    min: 0
                    max: 5
                    mode: slider
                    step: 1
              - name: show_unit
                label: Show Unit
                default: true
                selector:
                  boolean: {}
              - name: show_if_off
                label: Show When 'off'
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 6
            name: '5'
            schema:
              - name: entity
                label: Entity
                selector:
                  entity: {}
              - name: attribute
                label: Attribute (optional)
                selector:
                  attribute: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: decimals
                label: Decimal Places (for numbers)
                selector:
                  number:
                    min: 0
                    max: 5
                    mode: slider
                    step: 1
              - name: show_unit
                label: Show Unit
                default: true
                selector:
                  boolean: {}
              - name: show_if_off
                label: Show When 'off'
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
      - type: expandable
        title: Icon Badges
        icon: mdi:tag-outline
        name: icon_badges
        schema:
          - type: expandable
            title: Sub-Button 1
            name: '0'
            schema:
              - name: icon
                label: Icon
                selector:
                  icon: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 2
            name: '1'
            schema:
              - name: icon
                label: Icon
                selector:
                  icon: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 3
            name: '2'
            schema:
              - name: icon
                label: Icon
                selector:
                  icon: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 4
            name: '3'
            schema:
              - name: icon
                label: Icon
                selector:
                  icon: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 5
            name: '4'
            schema:
              - name: icon
                label: Icon
                selector:
                  icon: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 6
            name: '5'
            schema:
              - name: icon
                label: Icon
                selector:
                  icon: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
      - type: expandable
        title: Icon Badges EL1
        icon: mdi:tag-outline
        name: icon_badges_el1
        schema:
          - type: expandable
            title: Sub-Button 1
            name: '0'
            schema:
              - name: icon
                label: Icon
                selector:
                  icon: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 2
            name: '1'
            schema:
              - name: icon
                label: Icon
                selector:
                  icon: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 3
            name: '2'
            schema:
              - name: icon
                label: Icon
                selector:
                  icon: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 4
            name: '3'
            schema:
              - name: icon
                label: Icon
                selector:
                  icon: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 5
            name: '4'
            schema:
              - name: icon
                label: Icon
                selector:
                  icon: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 6
            name: '5'
            schema:
              - name: icon
                label: Icon
                selector:
                  icon: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
      - type: expandable
        title: Icon Badges EL2
        icon: mdi:tag-outline
        name: icon_badges_el2
        schema:
          - type: expandable
            title: Sub-Button 1
            name: '0'
            schema:
              - name: icon
                label: Icon
                selector:
                  icon: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 2
            name: '1'
            schema:
              - name: icon
                label: Icon
                selector:
                  icon: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 3
            name: '2'
            schema:
              - name: icon
                label: Icon
                selector:
                  icon: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 4
            name: '3'
            schema:
              - name: icon
                label: Icon
                selector:
                  icon: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 5
            name: '4'
            schema:
              - name: icon
                label: Icon
                selector:
                  icon: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 6
            name: '5'
            schema:
              - name: icon
                label: Icon
                selector:
                  icon: {}
              - name: color
                label: Badge Color
                selector:
                  ui_color: {}
              - name: text_color
                label: Text Color
                selector:
                  ui_color: {}
              - name: show_background
                label: Show Background
                default: true
                selector:
                  boolean: {}
              - name: scale
                label: Badge Size (scale)
                default: 1
                selector:
                  number:
                    min: 0.5
                    max: 3
                    mode: slider
                    step: 0.1
              - name: h_pos
                label: Horizontal Position (px)
                default: -8
                selector:
                  number:
                    min: -500
                    max: 500
                    mode: slider
                    step: 1
              - name: v_pos
                label: Vertical Position (px)
                default: -8
                selector:
                  number:
                    min: -100
                    max: 100
                    mode: slider
                    step: 1
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
      - type: expandable
        title: Main Icon Background
        icon: mdi:format-color-fill
        name: main_icon_background
        schema:
          - name: color
            label: Background Color
            selector:
              ui_color: {}
          - name: opacity
            label: Opacity
            default: 1
            selector:
              number:
                min: 0
                max: 1
                step: 0.05
                mode: slider
          - name: condition
            label: Condition (optional)
            selector:
              condition: {}
      - type: expandable
        title: Sub-Buttons Background
        icon: mdi:format-color-fill
        name: sub_buttons_background
        schema:
          - type: expandable
            title: Sub-Button 1
            name: '0'
            schema:
              - name: type
                label: Background Type
                selector:
                  select:
                    options:
                      - label: Solid
                        value: solid
                      - label: Two-Color Gradient
                        value: two-color
                      - label: Three-Color Gradient
                        value: three-color
              - name: color
                label: Solid Color
                selector:
                  ui_color:
                    include_none: true
              - name: color1
                label: Gradient Color 1
                selector:
                  ui_color: {}
              - name: color2
                label: Gradient Color 2
                selector:
                  ui_color: {}
              - name: color3
                label: Gradient Color 3
                selector:
                  ui_color: {}
              - name: opacity
                label: Background Opacity
                default: 1
                selector:
                  number:
                    min: 0
                    max: 1
                    step: 0.05
                    mode: slider
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 2
            name: '1'
            schema:
              - name: type
                label: Background Type
                selector:
                  select:
                    options:
                      - label: Solid
                        value: solid
                      - label: Two-Color Gradient
                        value: two-color
                      - label: Three-Color Gradient
                        value: three-color
              - name: color
                label: Solid Color
                selector:
                  ui_color:
                    include_none: true
              - name: color1
                label: Gradient Color 1
                selector:
                  ui_color: {}
              - name: color2
                label: Gradient Color 2
                selector:
                  ui_color: {}
              - name: color3
                label: Gradient Color 3
                selector:
                  ui_color: {}
              - name: opacity
                label: Background Opacity
                default: 1
                selector:
                  number:
                    min: 0
                    max: 1
                    step: 0.05
                    mode: slider
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 3
            name: '2'
            schema:
              - name: type
                label: Background Type
                selector:
                  select:
                    options:
                      - label: Solid
                        value: solid
                      - label: Two-Color Gradient
                        value: two-color
                      - label: Three-Color Gradient
                        value: three-color
              - name: color
                label: Solid Color
                selector:
                  ui_color:
                    include_none: true
              - name: color1
                label: Gradient Color 1
                selector:
                  ui_color: {}
              - name: color2
                label: Gradient Color 2
                selector:
                  ui_color: {}
              - name: color3
                label: Gradient Color 3
                selector:
                  ui_color: {}
              - name: opacity
                label: Background Opacity
                default: 1
                selector:
                  number:
                    min: 0
                    max: 1
                    step: 0.05
                    mode: slider
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 4
            name: '3'
            schema:
              - name: type
                label: Background Type
                selector:
                  select:
                    options:
                      - label: Solid
                        value: solid
                      - label: Two-Color Gradient
                        value: two-color
                      - label: Three-Color Gradient
                        value: three-color
              - name: color
                label: Solid Color
                selector:
                  ui_color:
                    include_none: true
              - name: color1
                label: Gradient Color 1
                selector:
                  ui_color: {}
              - name: color2
                label: Gradient Color 2
                selector:
                  ui_color: {}
              - name: color3
                label: Gradient Color 3
                selector:
                  ui_color: {}
              - name: opacity
                label: Background Opacity
                default: 1
                selector:
                  number:
                    min: 0
                    max: 1
                    step: 0.05
                    mode: slider
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 5
            name: '4'
            schema:
              - name: type
                label: Background Type
                selector:
                  select:
                    options:
                      - label: Solid
                        value: solid
                      - label: Two-Color Gradient
                        value: two-color
                      - label: Three-Color Gradient
                        value: three-color
              - name: color
                label: Solid Color
                selector:
                  ui_color:
                    include_none: true
              - name: color1
                label: Gradient Color 1
                selector:
                  ui_color: {}
              - name: color2
                label: Gradient Color 2
                selector:
                  ui_color: {}
              - name: color3
                label: Gradient Color 3
                selector:
                  ui_color: {}
              - name: opacity
                label: Background Opacity
                default: 1
                selector:
                  number:
                    min: 0
                    max: 1
                    step: 0.05
                    mode: slider
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 6
            name: '5'
            schema:
              - name: type
                label: Background Type
                selector:
                  select:
                    options:
                      - label: Solid
                        value: solid
                      - label: Two-Color Gradient
                        value: two-color
                      - label: Three-Color Gradient
                        value: three-color
              - name: color
                label: Solid Color
                selector:
                  ui_color:
                    include_none: true
              - name: color1
                label: Gradient Color 1
                selector:
                  ui_color: {}
              - name: color2
                label: Gradient Color 2
                selector:
                  ui_color: {}
              - name: color3
                label: Gradient Color 3
                selector:
                  ui_color: {}
              - name: opacity
                label: Background Opacity
                default: 1
                selector:
                  number:
                    min: 0
                    max: 1
                    step: 0.05
                    mode: slider
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
      - type: expandable
        title: Entity Badge Effects
        icon: mdi:tag-text
        name: entity_badge_effects
        schema:
          - type: expandable
            title: Sub-Button 1
            name: '0'
            schema:
              - name: effect
                label: Effect Type
                selector:
                  select:
                    options:
                      - label: Pulse (Scale)
                        value: pulse
                      - label: Gentle Rotation
                        value: rotate
                      - label: Glow Effect
                        value: glow
                      - label: Gentle Shake
                        value: shake
                      - label: Float Up & Down
                        value: float
                      - label: Zoom
                        value: zoom
                      - label: Heartbeat
                        value: heartbeat
                      - label: Spin 360
                        value: spin-360
                      - label: Flip
                        value: flip
                      - label: Vibrate
                        value: vibrate
              - name: color
                label: Effect Color
                selector:
                  ui_color: {}
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 2
            name: '1'
            schema:
              - name: effect
                label: Effect Type
                selector:
                  select:
                    options:
                      - label: Pulse (Scale)
                        value: pulse
                      - label: Gentle Rotation
                        value: rotate
                      - label: Glow Effect
                        value: glow
                      - label: Gentle Shake
                        value: shake
                      - label: Float Up & Down
                        value: float
                      - label: Zoom
                        value: zoom
                      - label: Heartbeat
                        value: heartbeat
                      - label: Spin 360
                        value: spin-360
                      - label: Flip
                        value: flip
                      - label: Vibrate
                        value: vibrate
              - name: color
                label: Effect Color
                selector:
                  ui_color: {}
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 3
            name: '2'
            schema:
              - name: effect
                label: Effect Type
                selector:
                  select:
                    options:
                      - label: Pulse (Scale)
                        value: pulse
                      - label: Gentle Rotation
                        value: rotate
                      - label: Glow Effect
                        value: glow
                      - label: Gentle Shake
                        value: shake
                      - label: Float Up & Down
                        value: float
                      - label: Zoom
                        value: zoom
                      - label: Heartbeat
                        value: heartbeat
                      - label: Spin 360
                        value: spin-360
                      - label: Flip
                        value: flip
                      - label: Vibrate
                        value: vibrate
              - name: color
                label: Effect Color
                selector:
                  ui_color: {}
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 4
            name: '3'
            schema:
              - name: effect
                label: Effect Type
                selector:
                  select:
                    options:
                      - label: Pulse (Scale)
                        value: pulse
                      - label: Gentle Rotation
                        value: rotate
                      - label: Glow Effect
                        value: glow
                      - label: Gentle Shake
                        value: shake
                      - label: Float Up & Down
                        value: float
                      - label: Zoom
                        value: zoom
                      - label: Heartbeat
                        value: heartbeat
                      - label: Spin 360
                        value: spin-360
                      - label: Flip
                        value: flip
                      - label: Vibrate
                        value: vibrate
              - name: color
                label: Effect Color
                selector:
                  ui_color: {}
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 5
            name: '4'
            schema:
              - name: effect
                label: Effect Type
                selector:
                  select:
                    options:
                      - label: Pulse (Scale)
                        value: pulse
                      - label: Gentle Rotation
                        value: rotate
                      - label: Glow Effect
                        value: glow
                      - label: Gentle Shake
                        value: shake
                      - label: Float Up & Down
                        value: float
                      - label: Zoom
                        value: zoom
                      - label: Heartbeat
                        value: heartbeat
                      - label: Spin 360
                        value: spin-360
                      - label: Flip
                        value: flip
                      - label: Vibrate
                        value: vibrate
              - name: color
                label: Effect Color
                selector:
                  ui_color: {}
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 6
            name: '5'
            schema:
              - name: effect
                label: Effect Type
                selector:
                  select:
                    options:
                      - label: Pulse (Scale)
                        value: pulse
                      - label: Gentle Rotation
                        value: rotate
                      - label: Glow Effect
                        value: glow
                      - label: Gentle Shake
                        value: shake
                      - label: Float Up & Down
                        value: float
                      - label: Zoom
                        value: zoom
                      - label: Heartbeat
                        value: heartbeat
                      - label: Spin 360
                        value: spin-360
                      - label: Flip
                        value: flip
                      - label: Vibrate
                        value: vibrate
              - name: color
                label: Effect Color
                selector:
                  ui_color: {}
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
      - type: expandable
        title: Icon Badge Effects
        icon: mdi:tag-outline
        name: icon_badge_effects
        schema:
          - type: expandable
            title: Sub-Button 1
            name: '0'
            schema:
              - name: effect
                label: Effect Type
                selector:
                  select:
                    options:
                      - label: Pulse (Scale)
                        value: pulse
                      - label: Gentle Rotation
                        value: rotate
                      - label: Glow Effect
                        value: glow
                      - label: Gentle Shake
                        value: shake
                      - label: Float Up & Down
                        value: float
                      - label: Zoom
                        value: zoom
                      - label: Heartbeat
                        value: heartbeat
                      - label: Spin 360
                        value: spin-360
                      - label: Flip
                        value: flip
                      - label: Vibrate
                        value: vibrate
              - name: color
                label: Effect Color
                selector:
                  ui_color: {}
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 2
            name: '1'
            schema:
              - name: effect
                label: Effect Type
                selector:
                  select:
                    options:
                      - label: Pulse (Scale)
                        value: pulse
                      - label: Gentle Rotation
                        value: rotate
                      - label: Glow Effect
                        value: glow
                      - label: Gentle Shake
                        value: shake
                      - label: Float Up & Down
                        value: float
                      - label: Zoom
                        value: zoom
                      - label: Heartbeat
                        value: heartbeat
                      - label: Spin 360
                        value: spin-360
                      - label: Flip
                        value: flip
                      - label: Vibrate
                        value: vibrate
              - name: color
                label: Effect Color
                selector:
                  ui_color: {}
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 3
            name: '2'
            schema:
              - name: effect
                label: Effect Type
                selector:
                  select:
                    options:
                      - label: Pulse (Scale)
                        value: pulse
                      - label: Gentle Rotation
                        value: rotate
                      - label: Glow Effect
                        value: glow
                      - label: Gentle Shake
                        value: shake
                      - label: Float Up & Down
                        value: float
                      - label: Zoom
                        value: zoom
                      - label: Heartbeat
                        value: heartbeat
                      - label: Spin 360
                        value: spin-360
                      - label: Flip
                        value: flip
                      - label: Vibrate
                        value: vibrate
              - name: color
                label: Effect Color
                selector:
                  ui_color: {}
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 4
            name: '3'
            schema:
              - name: effect
                label: Effect Type
                selector:
                  select:
                    options:
                      - label: Pulse (Scale)
                        value: pulse
                      - label: Gentle Rotation
                        value: rotate
                      - label: Glow Effect
                        value: glow
                      - label: Gentle Shake
                        value: shake
                      - label: Float Up & Down
                        value: float
                      - label: Zoom
                        value: zoom
                      - label: Heartbeat
                        value: heartbeat
                      - label: Spin 360
                        value: spin-360
                      - label: Flip
                        value: flip
                      - label: Vibrate
                        value: vibrate
              - name: color
                label: Effect Color
                selector:
                  ui_color: {}
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 5
            name: '4'
            schema:
              - name: effect
                label: Effect Type
                selector:
                  select:
                    options:
                      - label: Pulse (Scale)
                        value: pulse
                      - label: Gentle Rotation
                        value: rotate
                      - label: Glow Effect
                        value: glow
                      - label: Gentle Shake
                        value: shake
                      - label: Float Up & Down
                        value: float
                      - label: Zoom
                        value: zoom
                      - label: Heartbeat
                        value: heartbeat
                      - label: Spin 360
                        value: spin-360
                      - label: Flip
                        value: flip
                      - label: Vibrate
                        value: vibrate
              - name: color
                label: Effect Color
                selector:
                  ui_color: {}
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 6
            name: '5'
            schema:
              - name: effect
                label: Effect Type
                selector:
                  select:
                    options:
                      - label: Pulse (Scale)
                        value: pulse
                      - label: Gentle Rotation
                        value: rotate
                      - label: Glow Effect
                        value: glow
                      - label: Gentle Shake
                        value: shake
                      - label: Float Up & Down
                        value: float
                      - label: Zoom
                        value: zoom
                      - label: Heartbeat
                        value: heartbeat
                      - label: Spin 360
                        value: spin-360
                      - label: Flip
                        value: flip
                      - label: Vibrate
                        value: vibrate
              - name: color
                label: Effect Color
                selector:
                  ui_color: {}
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
      - type: expandable
        title: Icon Effects
        icon: mdi:weather-windy-variant
        name: icon_effects
        schema:
          - type: expandable
            title: Main Icon
            name: main_icon
            schema:
              - name: effect
                label: Effect Type
                selector:
                  select:
                    options:
                      - label: Pulse (Scale)
                        value: pulse
                      - label: Gentle Rotation
                        value: rotate
                      - label: Glow Effect
                        value: glow
                      - label: Gentle Shake
                        value: shake
                      - label: Float Up & Down
                        value: float
                      - label: Zoom
                        value: zoom
                      - label: Heartbeat
                        value: heartbeat
                      - label: Spin 360
                        value: spin-360
                      - label: Flip
                        value: flip
                      - label: Vibrate
                        value: vibrate
              - name: color
                label: Effect Color
                selector:
                  ui_color: {}
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button Icon Effects
            name: sub_button_icon
            schema:
              - type: expandable
                title: Sub-Button Icon 1
                name: '0'
                schema:
                  - name: effect
                    label: Effect Type
                    selector:
                      select:
                        options:
                          - label: Pulse (Scale)
                            value: pulse
                          - label: Gentle Rotation
                            value: rotate
                          - label: Glow Effect
                            value: glow
                          - label: Gentle Shake
                            value: shake
                          - label: Float Up & Down
                            value: float
                          - label: Zoom
                            value: zoom
                          - label: Heartbeat
                            value: heartbeat
                          - label: Spin 360
                            value: spin-360
                          - label: Flip
                            value: flip
                          - label: Vibrate
                            value: vibrate
                  - name: color
                    label: Effect Color
                    selector:
                      ui_color: {}
                  - name: condition
                    label: Condition (optional)
                    selector:
                      condition: {}
              - type: expandable
                title: Sub-Button Icon 2
                name: '1'
                schema:
                  - name: effect
                    label: Effect Type
                    selector:
                      select:
                        options:
                          - label: Pulse (Scale)
                            value: pulse
                          - label: Gentle Rotation
                            value: rotate
                          - label: Glow Effect
                            value: glow
                          - label: Gentle Shake
                            value: shake
                          - label: Float Up & Down
                            value: float
                          - label: Zoom
                            value: zoom
                          - label: Heartbeat
                            value: heartbeat
                          - label: Spin 360
                            value: spin-360
                          - label: Flip
                            value: flip
                          - label: Vibrate
                            value: vibrate
                  - name: color
                    label: Effect Color
                    selector:
                      ui_color: {}
                  - name: condition
                    label: Condition (optional)
                    selector:
                      condition: {}
              - type: expandable
                title: Sub-Button Icon 3
                name: '2'
                schema:
                  - name: effect
                    label: Effect Type
                    selector:
                      select:
                        options:
                          - label: Pulse (Scale)
                            value: pulse
                          - label: Gentle Rotation
                            value: rotate
                          - label: Glow Effect
                            value: glow
                          - label: Gentle Shake
                            value: shake
                          - label: Float Up & Down
                            value: float
                          - label: Zoom
                            value: zoom
                          - label: Heartbeat
                            value: heartbeat
                          - label: Spin 360
                            value: spin-360
                          - label: Flip
                            value: flip
                          - label: Vibrate
                            value: vibrate
                  - name: color
                    label: Effect Color
                    selector:
                      ui_color: {}
                  - name: condition
                    label: Condition (optional)
                    selector:
                      condition: {}
              - type: expandable
                title: Sub-Button Icon 4
                name: '3'
                schema:
                  - name: effect
                    label: Effect Type
                    selector:
                      select:
                        options:
                          - label: Pulse (Scale)
                            value: pulse
                          - label: Gentle Rotation
                            value: rotate
                          - label: Glow Effect
                            value: glow
                          - label: Gentle Shake
                            value: shake
                          - label: Float Up & Down
                            value: float
                          - label: Zoom
                            value: zoom
                          - label: Heartbeat
                            value: heartbeat
                          - label: Spin 360
                            value: spin-360
                          - label: Flip
                            value: flip
                          - label: Vibrate
                            value: vibrate
                  - name: color
                    label: Effect Color
                    selector:
                      ui_color: {}
                  - name: condition
                    label: Condition (optional)
                    selector:
                      condition: {}
              - type: expandable
                title: Sub-Button Icon 5
                name: '4'
                schema:
                  - name: effect
                    label: Effect Type
                    selector:
                      select:
                        options:
                          - label: Pulse (Scale)
                            value: pulse
                          - label: Gentle Rotation
                            value: rotate
                          - label: Glow Effect
                            value: glow
                          - label: Gentle Shake
                            value: shake
                          - label: Float Up & Down
                            value: float
                          - label: Zoom
                            value: zoom
                          - label: Heartbeat
                            value: heartbeat
                          - label: Spin 360
                            value: spin-360
                          - label: Flip
                            value: flip
                          - label: Vibrate
                            value: vibrate
                  - name: color
                    label: Effect Color
                    selector:
                      ui_color: {}
                  - name: condition
                    label: Condition (optional)
                    selector:
                      condition: {}
              - type: expandable
                title: Sub-Button Icon 6
                name: '5'
                schema:
                  - name: effect
                    label: Effect Type
                    selector:
                      select:
                        options:
                          - label: Pulse (Scale)
                            value: pulse
                          - label: Gentle Rotation
                            value: rotate
                          - label: Glow Effect
                            value: glow
                          - label: Gentle Shake
                            value: shake
                          - label: Float Up & Down
                            value: float
                          - label: Zoom
                            value: zoom
                          - label: Heartbeat
                            value: heartbeat
                          - label: Spin 360
                            value: spin-360
                          - label: Flip
                            value: flip
                          - label: Vibrate
                            value: vibrate
                  - name: color
                    label: Effect Color
                    selector:
                      ui_color: {}
                  - name: condition
                    label: Condition (optional)
                    selector:
                      condition: {}
      - type: expandable
        title: Sub-Button Effects
        icon: mdi:animation-outline
        name: sub_button_effects
        schema:
          - type: expandable
            title: Sub-Button 1
            name: '0'
            schema:
              - name: effect
                label: Effect Type
                selector:
                  select:
                    options:
                      - label: Pulse (Scale)
                        value: pulse
                      - label: Gentle Rotation
                        value: rotate
                      - label: Glow Effect
                        value: glow
                      - label: Gentle Shake
                        value: shake
                      - label: Float Up & Down
                        value: float
                      - label: Zoom
                        value: zoom
                      - label: Heartbeat
                        value: heartbeat
                      - label: Spin 360
                        value: spin-360
                      - label: Flip
                        value: flip
                      - label: Vibrate
                        value: vibrate
              - name: color
                label: Effect Color (e.g. for Glow)
                selector:
                  ui_color: {}
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 2
            name: '1'
            schema:
              - name: effect
                label: Effect Type
                selector:
                  select:
                    options:
                      - label: Pulse (Scale)
                        value: pulse
                      - label: Gentle Rotation
                        value: rotate
                      - label: Glow Effect
                        value: glow
                      - label: Gentle Shake
                        value: shake
                      - label: Float Up & Down
                        value: float
                      - label: Zoom
                        value: zoom
                      - label: Heartbeat
                        value: heartbeat
                      - label: Spin 360
                        value: spin-360
                      - label: Flip
                        value: flip
                      - label: Vibrate
                        value: vibrate
              - name: color
                label: Effect Color (e.g. for Glow)
                selector:
                  ui_color: {}
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 3
            name: '2'
            schema:
              - name: effect
                label: Effect Type
                selector:
                  select:
                    options:
                      - label: Pulse (Scale)
                        value: pulse
                      - label: Gentle Rotation
                        value: rotate
                      - label: Glow Effect
                        value: glow
                      - label: Gentle Shake
                        value: shake
                      - label: Float Up & Down
                        value: float
                      - label: Zoom
                        value: zoom
                      - label: Heartbeat
                        value: heartbeat
                      - label: Spin 360
                        value: spin-360
                      - label: Flip
                        value: flip
                      - label: Vibrate
                        value: vibrate
              - name: color
                label: Effect Color (e.g. for Glow)
                selector:
                  ui_color: {}
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 4
            name: '3'
            schema:
              - name: effect
                label: Effect Type
                selector:
                  select:
                    options:
                      - label: Pulse (Scale)
                        value: pulse
                      - label: Gentle Rotation
                        value: rotate
                      - label: Glow Effect
                        value: glow
                      - label: Gentle Shake
                        value: shake
                      - label: Float Up & Down
                        value: float
                      - label: Zoom
                        value: zoom
                      - label: Heartbeat
                        value: heartbeat
                      - label: Spin 360
                        value: spin-360
                      - label: Flip
                        value: flip
                      - label: Vibrate
                        value: vibrate
              - name: color
                label: Effect Color (e.g. for Glow)
                selector:
                  ui_color: {}
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 5
            name: '4'
            schema:
              - name: effect
                label: Effect Type
                selector:
                  select:
                    options:
                      - label: Pulse (Scale)
                        value: pulse
                      - label: Gentle Rotation
                        value: rotate
                      - label: Glow Effect
                        value: glow
                      - label: Gentle Shake
                        value: shake
                      - label: Float Up & Down
                        value: float
                      - label: Zoom
                        value: zoom
                      - label: Heartbeat
                        value: heartbeat
                      - label: Spin 360
                        value: spin-360
                      - label: Flip
                        value: flip
                      - label: Vibrate
                        value: vibrate
              - name: color
                label: Effect Color (e.g. for Glow)
                selector:
                  ui_color: {}
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
          - type: expandable
            title: Sub-Button 6
            name: '5'
            schema:
              - name: effect
                label: Effect Type
                selector:
                  select:
                    options:
                      - label: Pulse (Scale)
                        value: pulse
                      - label: Gentle Rotation
                        value: rotate
                      - label: Glow Effect
                        value: glow
                      - label: Gentle Shake
                        value: shake
                      - label: Float Up & Down
                        value: float
                      - label: Zoom
                        value: zoom
                      - label: Heartbeat
                        value: heartbeat
                      - label: Spin 360
                        value: spin-360
                      - label: Flip
                        value: flip
                      - label: Vibrate
                        value: vibrate
              - name: color
                label: Effect Color (e.g. for Glow)
                selector:
                  ui_color: {}
              - name: condition
                label: Condition (optional)
                selector:
                  condition: {}
      
```

</details>

---

### Screenshot:

Important: The first screenshot here will be used on the Module Store, so please provide one.
