# Background

Between 2011 and 2020 I was working on [Substance.js](https://github.com/substance/substance), a JavaScript library for web-based content editing. We also built [Texture](https://elifesciences.org/labs/8de87c33/texture-an-open-science-manuscript-editor), a visual editor for scientific manuscripts (think LaTeX but with WYSIWYG editing).

Since summer 2024, I'm spending more than 50% of my time developing [Svedit](https://svedit.dev), a lightweight library designed to enable building [in-place editable websites](https://editable.website) I'm building this directly on top of web API's, rather than using an existing library (ProseMirror, Lexical,...).

# Findings

Compared to pre 2020, the Situation got much better with introduction of the `beforeinput` and new API's that are part of [Input Events Level 2 spec](https://www.w3.org/TR/input-events-2/). Currently, I'm able to handle almost all inputs myself, apply a change to my internal model, which then triggers an incremental rerender, updating the DOM accordingly.

What still gives me troubles is handling composition events. I have some workarounds in place, such as disabling incremental rendering, but I'd hope this could be done in a better way.

# Top 4 Requests

## Ability to rollback DOM changes of a composition at oncompositionend stage

```js
function oncompositionend(event) {
	// Make sure the DOM is in the exact same state as before the composition started
  event.resetDOM();
  // Apply the finished composition to the document
  doc.apply(doc.tr.insert_text(event.data));
  // Incremental reactive-rerendering can take place safely
}
```

This is just a simplified example. See [#13](https://github.com/michael/web-editing/issues/13).

## Ability to capture event.getTargetRanges() at oncompositionstart stage

Currently event.getTargetRanges() can only be captured at the onbeforeinput stage, which is not just inconvenient, but also unsufficient in edge cases where (e.g. turn dictation on and off on Samsung-Android)

See [#11](https://github.com/michael/web-editing/issues/11).

## Ability to cancel composition events at the very beginning

I found a workaround, that prevents my editor to crash when you start a composition for a given selection.

```js
function oncompositionstart(event) {
  // eg. when multiple nodes are selected, or inside a cursor trap
  if (doc.selection.type !== 'text') {
    const dom_selection = window.getSelection();
    dom_selection.removeAllRanges();
  } else {
    // handle the IME, when you have text cursor/selection.
  }
}
```

See [#6](https://github.com/michael/web-editing/issues/6).

## Ability create and handle custom history events

The problem is that when I handle input myself (onbeforeinput with event.preventDefault()) no history entries are created. Hence when you use the browser's native undo (e.g. Edit > Undo) or other triggers/shortcuts, they don't fire historyUndo/historyRedo events.

In order to allow not just rich text editors, but any apps (e.g. Figma) utilize the native history events, an API for creating and handling custom history events is needed.

See [#1](https://github.com/michael/web-editing/issues/1).
