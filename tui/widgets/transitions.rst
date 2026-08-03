TransitionWidget
=================

``TransitionWidget`` blends between two widgets over time using one of several built-in
cinematic effects, or a custom one. It works as a full-screen swap or scoped to a single widget
inside a layout.

When to Use
-----------

Use ``TransitionWidget`` for a wizard, a slide deck, a "press space to continue" demo, or a
game's menu-to-level change: anywhere a screen change should animate instead of cutting
instantly. Two placements are common:

* **Full-screen.** The ``TransitionWidget`` is the app's root; every screen change goes through
  it.
* **Scoped to one widget.** The rest of the layout (sidebar, header, status bar) stays put, and
  only one panel changes content with a transition.

Basic Usage
-----------

.. code-block:: php

    use Symfony\Component\Tui\Transition\SlideTransition;
    use Symfony\Component\Tui\Transition\TransitionDirection;
    use Symfony\Component\Tui\Transition\TransitionWidget;

    $stage = new TransitionWidget();
    $stage->expandVertically(true);
    $tui->add($stage);

    $stage->start($screenA, $screenB, new SlideTransition(TransitionDirection::Left), 0.5);

``start()`` takes the current widget, the destination widget, a transition strategy and a
duration in seconds. Once the ``TransitionWidget`` is attached to a running Tui, it schedules
its own ticks and advances the animation automatically (no manual ``tick()`` call or
``Tui::onTick()`` wiring needed).

.. tip::

    ``start()`` does not recreate your widgets: it animates between the two instances you pass
    in and keeps them. A counter keeps its value, an input keeps its text, a list keeps its
    selection. Reuse the same instance across calls to preserve state; build a fresh widget only
    when you want to reset it.

Chaining Transitions
---------------------

Call ``start()`` again to move to a new screen. ``getTo()`` always returns whatever the last
call's destination was, so it doubles as the next call's source::

    $stage->start($stage->getTo(), $screenC, new WipeTransition(TransitionDirection::Right), 0.4);

Build a fresh transition instance on every call if the direction, easing or duration needs to
change between navigations: transitions are cheap, stateless value objects.

Direction
---------

Most transitions take a ``TransitionDirection`` in their constructor. Each transition only
accepts the subset that makes sense for it; passing an unsupported case throws an
``InvalidArgumentException`` at construction:

- ``Left``, ``Right``, ``Top``, ``Bottom``: ``SlideTransition``, ``WipeTransition``.
- ``Horizontal``, ``Vertical``: ``SliceTransition``, ``ShuttersTransition``.
- ``In``, ``Out``: ``DiamondTransition``, ``SpiralTransition``.

::

    use Symfony\Component\Tui\Transition\TransitionDirection;

    $slide = new SlideTransition(TransitionDirection::Top);

Easing
------

Pass any ``callable(float): float`` to ``setEasing()`` to change the pacing of a transition. It
receives the linear progress (``0.0`` to ``1.0``) and returns the adjusted value::

    $transition = (new SlideTransition())->setEasing(fn (float $p) => $p * $p); // ease-in

Linear (no easing) by default. A bouncy or elastic easing that overshoots past ``0.0``/``1.0``
mid-transition is supported: only the exact boundary progress reported by the widget itself
(the transition's true start and end) short-circuits to the source/destination screen; every
other value, including an overshoot, is passed through your easing function and rendered.

Size
----

``SliceTransition``, ``ShuttersTransition`` and ``SpiralTransition`` take a second constructor
argument controlling the granularity of the effect, in terminal cells: band thickness for the
first two, block size for ``SpiralTransition``. Larger values give a chunkier animation that is
also cheaper to compute per frame::

    new ShuttersTransition(TransitionDirection::Horizontal, 2);

Built-in Transitions
---------------------

Snapshots below show progress stages from screen A (``■``) to screen B (``□``).

Slide
~~~~~

Pushes the outgoing screen off-frame while the new one slides in behind it::

    new SlideTransition(TransitionDirection::Right);

.. code-block:: text

    □□□■■■■■■■■■■■■   □□□□□□■■■■■■■■■   □□□□□□□□□□■■■■■   □□□□□□□□□□□□□□■
    □□□■■■■■■■■■■■■   □□□□□□■■■■■■■■■   □□□□□□□□□□■■■■■   □□□□□□□□□□□□□□■
    □□□■■■■■■■■■■■■   □□□□□□■■■■■■■■■   □□□□□□□□□□■■■■■   □□□□□□□□□□□□□□■
    □□□■■■■■■■■■■■■   □□□□□□■■■■■■■■■   □□□□□□□□□□■■■■■   □□□□□□□□□□□□□□■
    □□□■■■■■■■■■■■■   □□□□□□■■■■■■■■■   □□□□□□□□□□■■■■■   □□□□□□□□□□□□□□■

Wipe
~~~~

Reveals the new screen from one edge, like a curtain, without moving the old one::

    new WipeTransition(TransitionDirection::Bottom);

.. code-block:: text

    □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□
    ■■■■■■■■■■■■■■■   □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□
    ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■   □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□
    ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■   □□□□□□□□□□□□□□□
    ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■

Slice
~~~~~

Alternates opposing bands (``Horizontal``) or reveals adjustable-width columns (``Vertical``)::

    new SliceTransition(TransitionDirection::Horizontal, sliceSize: 2);

.. code-block:: text

    □□□■■■■■■■■■■■■   □□□□□□□■■■■■■■■   □□□□□□□□□□□■■■■   □□□□□□□□□□□□□□□
    ■■■■■■■■■■■■□□□   ■■■■■■■■□□□□□□□   ■■■■□□□□□□□□□□□   □□□□□□□□□□□□□□□
    □□□■■■■■■■■■■■■   □□□□□□□■■■■■■■■   □□□□□□□□□□□■■■■   □□□□□□□□□□□□□□□
    ■■■■■■■■■■■■□□□   ■■■■■■■■□□□□□□□   ■■■■□□□□□□□□□□□   □□□□□□□□□□□□□□□
    □□□■■■■■■■■■■■■   □□□□□□□■■■■■■■■   □□□□□□□□□□□■■■■   □□□□□□□□□□□□□□□

Diamond
~~~~~~~

The new screen grows from, or shrinks into, the center in a diamond shape::

    new DiamondTransition(TransitionDirection::Out);

.. code-block:: text

    ■■■■■■■■■■■■■■■   ■■■■□□□□□□■■■■■   ■■□□□□□□□□□□■■■   □□□□□□□□□□□□□□■
    ■■■■■■□□■■■■■■■   ■■■□□□□□□□□■■■■   ■□□□□□□□□□□□□■■   □□□□□□□□□□□□□□□
    ■■■■■□□□□■■■■■■   ■■□□□□□□□□□□■■■   □□□□□□□□□□□□□□■   □□□□□□□□□□□□□□□
    ■■■■■■□□■■■■■■■   ■■■□□□□□□□□■■■■   ■□□□□□□□□□□□□■■   □□□□□□□□□□□□□□□
    ■■■■■■■■■■■■■■■   ■■■■□□□□□□■■■■■   ■■□□□□□□□□□□■■■   □□□□□□□□□□□□□□■

``TransitionDirection::In`` is the mirror: a shrinking diamond of screen A disappears into
screen B instead.

Shutters
~~~~~~~~

Evenly spaced slats reveal the new screen in alternating bands::

    new ShuttersTransition(TransitionDirection::Horizontal, size: 2);

.. code-block:: text

    □□□□□□■■■■■■■■■   □□□□□□□□□□□□□■■   □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□
    ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■   □□□□□■■■■■■■■■■   □□□□□□□□□□□□□■■
    □□□□□□■■■■■■■■■   □□□□□□□□□□□□□■■   □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□
    ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■   □□□□□■■■■■■■■■■   □□□□□□□□□□□□□■■
    □□□□□□■■■■■■■■■   □□□□□□□□□□□□□■■   □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□

Spiral
~~~~~~

The new screen unfolds block by block along a spiral path::

    new SpiralTransition(TransitionDirection::Out, size: 3);

.. code-block:: text

    ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■   ■■■■■■■■■□□□□□□   ■■■□□□□□□□□□□□□
    ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■   ■■■■■■■■■□□□□□□   ■■■□□□□□□□□□□□□
    ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■   ■■■■■■■■■□□□□□□   ■■■□□□□□□□□□□□□
    □□□□□□■■■■■■■■■   □□□□□□□□□□□□■■■   □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□
    □□□□□□■■■■■■■■■   □□□□□□□□□□□□■■■   □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□

Custom Transitions
--------------------

Extend ``AbstractTransition`` and implement ``doBlend()``::

    use Symfony\Component\Tui\Transition\AbstractTransition;

    final class FadeToBlackTransition extends AbstractTransition
    {
        protected function doBlend(array $fromLines, array $toLines, int $width, int $height, float $progress): array
        {
            return $progress < 0.5 ? $fromLines : $toLines;
        }
    }

``$fromLines``/``$toLines`` are the two screens as ANSI-formatted line arrays, one string per
row. ``$progress`` is already eased. The widget's own true start and end (``progress`` exactly
``0.0`` or ``1.0``) are short-circuited by ``AbstractTransition`` to the source/destination
screen before ``doBlend()`` is ever called, so it only runs for values in between. Pass an
instance to ``start()`` exactly like a built-in transition.

Styling
-------

Like any widget, ``TransitionWidget`` accepts padding, borders and a background through the
Style system (see :doc:`/tui/style/index`). What you actually see during a transition, though,
is driven by the ``from``/``to`` widgets themselves: style ``TransitionWidget`` only for
things like a container border around the whole stage.

Events
------

``TransitionWidget`` does not emit any events.
