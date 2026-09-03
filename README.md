# Explainer: `scroll-snap-stop: before`

## Introduction

In [CSS Scroll Snap Module Level 1](https://drafts.csswg.org/css-scroll-snap-1/), the `scroll-snap-stop` property controls whether a scroll container passes over snap positions during inertial scrolling.

This explainer proposes the `before` value for `scroll-snap-stop`. When an inertial scroll (such as a fling) encounters a snap target marked `scroll-snap-stop: before`, scrolling stops and snaps to the snap target immediately before the specified element.

## Goals & Non-Goals

### Goals
- Enable developers to declaratively stop inertial scrolling at the snap target immediately before a designated element.
- Support common web patterns such as pull-to-refresh indicators, paywall boundaries, and interstitial barriers without JavaScript scroll interception.
- Integrate seamlessly with existing `scroll-snap-type` and `scroll-snap-align` behaviors.

### Non-Goals
- Alter direct, continuous touch/pointer dragging past the snap area while the user maintains contact.
- Modify programmatic scrolling APIs (e.g., `Element.scrollTo()`).

## Background and Motivation

Web interfaces often require scrollable containers to stop ahead of specific elements rather than on top of them. A classic example is a "pull-to-refresh" indicator situated above a feed:

1. The scroll container defaults its scroll position to the start of the feed content, keeping the pull-to-refresh indicator hidden off-screen above the visible snapport.
2. When a user scrolls down through the feed and subsequently flings upward with high momentum, they expect the scroll to halt at the top of the feed content rather than triggering an accidental refresh.
3. To actually initiate a refresh, the user must deliberately drag downward past the feed boundary to bring the indicator into view.

Existing `scroll-snap-stop` values cannot achieve this declarative balance:
- `normal` allows fast upward flings to bypass the feed start and land on or overshoot the indicator.
- `always` forces flings to snap onto the indicator itself whenever scrolling upward past the feed start.

`scroll-snap-stop: before` provides the necessary primitive by preventing inertial flings from reaching the indicator, snapping instead to the feed content start directly before it.

## Proposal

The `before` keyword is added to `scroll-snap-stop`:

```css
scroll-snap-stop: before;
```

### Behavior

During inertial scrolling (such as a fling gesture):

- When the scroll trajectory reaches or approaches a snap area with `scroll-snap-stop: before`, the scroll container cannot pass over that element.
- Instead of snapping onto the `before` element itself, the container snaps to the snap target immediately before it in the direction of the scroll.

## Use Case: Pull to Refresh ([Demo](https://htmlpreview.github.io/?https://github.com/explainers-by-googlers/scroll-snap-stop-before/blob/main/pull_to_refresh_demo.html))

In a vertical feed with pull-to-refresh functionality, the container defines two primary snap positions:
- A pull-to-refresh header located at the top of the scrollable area.
- The main feed content container located immediately below the header.

By applying `scroll-snap-stop: before` to the pull-to-refresh header and standard snap alignment to the main feed start, the following behavior is achieved:

- **Upward Momentum Flings**: Fast upward flings through the feed halt and snap at the main feed start, leaving the refresh indicator unreached and untriggered.
- **Intentional Drag**: When the user deliberately drags downward from the top of the feed with their finger down, they can pull the refresh header into view to trigger the action.

```css
.scroller {
  overflow-y: scroll;
  scroll-snap-type: y mandatory;
}

.ptr-indicator {
  scroll-snap-align: start;
  scroll-snap-stop: before;
}

.feed-content {
  scroll-snap-align: start;
}
```

## Compatibility

- **Default Behavior**: Elements without `scroll-snap-stop: before` retain the initial value `scroll-snap-stop: normal`.
- **Fallback**: Browsers without support for `before` treat the value as invalid and fall back to default snapping behavior.
