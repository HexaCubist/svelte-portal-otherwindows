<script context="module">
  import { tick } from "svelte";

  /**
   * Usage: <div use:portal={'css selector'}> or <div use:portal={document.body}> or <div use:portal={document.body} window={otherwindowcontext}>
   *
   * @param {HTMLElement} el
   * @param {HTMLElement|string} target DOM Element or CSS Selector
   * @param {Window} [targetWindow] Target window DOM to use for globals. Required for cross-iframe portals
   */
  export function portal(el, target = "body", targetWindow = undefined) {
    let targetEl;
    async function update(newTarget) {
      target = newTarget;
      if (typeof target === "string") {
        targetEl = document.querySelector(target);
        if (targetEl === null) {
          await tick();
          targetEl = document.querySelector(target);
        }
        if (targetEl === null) {
          throw new Error(
            `No element found matching css selector: "${target}"`
          );
        }
      } else if (!targetWindow && target instanceof HTMLElement) {
        targetEl = target;
      } else if (targetWindow && target instanceof targetWindow.HTMLElement) {
        targetEl = target;
      } else {
        throw new TypeError(
          `Unknown portal target type: ${
            target === null ? "null" : typeof target
          }. Allowed types: string (CSS selector) or HTMLElement.`
        );
      }
      targetEl.appendChild(el);
      el.hidden = false;
    }

    function destroy() {
      if (el.parentNode) {
        el.parentNode.removeChild(el);
      }
    }

    update(target);
    return {
      update,
      destroy,
    };
  }
</script>

<script>
  /**
   * DOM Element or CSS Selector
   * @type { HTMLElement|string}
   */
  export let target = "body";
</script>

<div use:portal={target} hidden>
  <slot />
</div>
