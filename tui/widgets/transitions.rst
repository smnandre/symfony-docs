TransitionWidget
================

``TransitionWidget`` blends between two widgets over time using one of six
built-in cinematic effects or a custom one. It works as a full-screen
swap or scoped to a single widget inside a layout.

When to Use
-----------

Use ``TransitionWidget`` for a wizard, a slide deck, a "press space to
continue" demo or a game's menu-to-level change: anywhere a screen change
should animate instead of cutting instantly. Two placements are common:

* **Full-screen.** The ``TransitionWidget`` is the app's root; every screen
  change goes through it.
* **Scoped to one widget.** The rest of the layout (sidebar, header, status
  bar) stays put and only one panel changes content with a transition.

Basic Usage
-----------

::

    use Symfony\Component\Tui\Transition\Direction\HorizontalDirection;
    use Symfony\Component\Tui\Transition\SlideTransition;
    use Symfony\Component\Tui\Widget\TransitionWidget;

    $stage = new TransitionWidget();
    $stage->expandVertically(true);
    $tui->add($stage);

    $stage->start(
        $screenA,
        $screenB,
        new SlideTransition(HorizontalDirection::Left),
        0.5,
    );

``start()`` takes the current widget, the destination widget, a transition
strategy and a duration in seconds. The duration must be greater than zero.
Once the ``TransitionWidget`` is attached to a running Tui, it schedules its
own ticks and advances the animation automatically. You don't need to call
``tick()`` manually or wire ``Tui::onTick()``.

.. tip::

    ``start()`` does not recreate your widgets: it animates between the two
    instances you pass in and keeps them. A counter keeps its value, an input
    keeps its text and a list keeps its selection. Reuse the same instance
    across calls to preserve state; build a fresh widget only when you want to
    reset it.

Chaining Transitions
--------------------

Call ``start()`` again to move to a new screen. ``getTo()`` always returns the
last call's destination, so it can be the next call's source::

    use Symfony\Component\Tui\Transition\WipeTransition;

    $stage->start(
        $stage->getTo(),
        $screenC,
        new WipeTransition(HorizontalDirection::Right),
        0.4,
    );

Direction and size are fixed when you construct a transition. Easing is stored
on the transition instance. Use separate instances when these settings differ
between navigations.

Direction
---------

Transitions use three direction enums:

* ``HorizontalDirection::Left`` and ``HorizontalDirection::Right`` select a
  horizontal entrance edge.
* ``VerticalDirection::Top`` and ``VerticalDirection::Bottom`` select a
  vertical entrance edge.
* ``RadialDirection::Inward`` and ``RadialDirection::Outward`` select whether
  a radial reveal travels toward or away from the center.

``SlideTransition``, ``WipeTransition``, ``SliceTransition`` and
``ShuttersTransition`` accept either ``HorizontalDirection`` or
``VerticalDirection``. ``DiamondTransition`` and ``SpiralTransition`` accept
``RadialDirection``. The constructor types let static analysis reject an
incompatible direction.

::

    use Symfony\Component\Tui\Transition\Direction\VerticalDirection;

    $slide = new SlideTransition(VerticalDirection::Top);

Easing
------

Pass any ``callable(float): float`` to ``setEasing()`` to change the pacing of
a transition. It receives the linear progress (``0.0`` to ``1.0``) and returns
the adjusted value::

    $transition = (new SlideTransition())->setEasing(
        fn (float $progress) => $progress * $progress,
    );

Progress is linear by default. A bouncy or elastic easing may overshoot past
``0.0`` or ``1.0`` during the transition. ``AbstractTransition`` applies the
easing and clamps the result before rendering. Only the widget's exact start
and end progress skip easing and return the complete source or destination
screen.

Size
----

``SliceTransition``, ``ShuttersTransition`` and ``SpiralTransition`` take a
``size`` constructor argument that controls the effect's granularity in
terminal cells. It sets the band thickness for the first two transitions and
the block size for ``SpiralTransition``. For ``SliceTransition``, ``size``
applies only to entrances from the top or bottom. The value must be greater
than zero::

    new SliceTransition(VerticalDirection::Top, size: 2);

Built-in Transitions
--------------------

The snapshots below show progress stages from screen A (``■``) to screen B
(``□``).

Slide
~~~~~

Pushes the outgoing screen off-frame while the new one slides in behind it::

    new SlideTransition(HorizontalDirection::Left);

.. code-block:: text

    □□□■■■■■■■■■■■■   □□□□□□■■■■■■■■■   □□□□□□□□□□■■■■■   □□□□□□□□□□□□□□■
    □□□■■■■■■■■■■■■   □□□□□□■■■■■■■■■   □□□□□□□□□□■■■■■   □□□□□□□□□□□□□□■
    □□□■■■■■■■■■■■■   □□□□□□■■■■■■■■■   □□□□□□□□□□■■■■■   □□□□□□□□□□□□□□■
    □□□■■■■■■■■■■■■   □□□□□□■■■■■■■■■   □□□□□□□□□□■■■■■   □□□□□□□□□□□□□□■
    □□□■■■■■■■■■■■■   □□□□□□■■■■■■■■■   □□□□□□□□□□■■■■■   □□□□□□□□□□□□□□■

Wipe
~~~~

Reveals the new screen from one edge, like a curtain, without moving the old
one::

    new WipeTransition(VerticalDirection::Top);

.. code-block:: text

    □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□
    ■■■■■■■■■■■■■■■   □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□
    ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■   □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□
    ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■   □□□□□□□□□□□□□□□
    ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■

Slice
~~~~~

Reveals alternating rows from the left or right, or alternating column bands
from the top or bottom::

    new SliceTransition(HorizontalDirection::Left);

.. code-block:: text

    □□□■■■■■■■■■■■■   □□□□□□□■■■■■■■■   □□□□□□□□□□□■■■■   □□□□□□□□□□□□□□□
    ■■■■■■■■■■■■□□□   ■■■■■■■■□□□□□□□   ■■■■□□□□□□□□□□□   □□□□□□□□□□□□□□□
    □□□■■■■■■■■■■■■   □□□□□□□■■■■■■■■   □□□□□□□□□□□■■■■   □□□□□□□□□□□□□□□
    ■■■■■■■■■■■■□□□   ■■■■■■■■□□□□□□□   ■■■■□□□□□□□□□□□   □□□□□□□□□□□□□□□
    □□□■■■■■■■■■■■■   □□□□□□□■■■■■■■■   □□□□□□□□□□□■■■■   □□□□□□□□□□□□□□□

Diamond
~~~~~~~

The new screen grows from, or shrinks into, the center in a diamond shape::

    use Symfony\Component\Tui\Transition\Direction\RadialDirection;

    new DiamondTransition(RadialDirection::Outward);

.. code-block:: text

    ■■■■■■■■■■■■■■■   ■■■■□□□□□□■■■■■   ■■□□□□□□□□□□■■■   □□□□□□□□□□□□□□■
    ■■■■■■□□■■■■■■■   ■■■□□□□□□□□■■■■   ■□□□□□□□□□□□□■■   □□□□□□□□□□□□□□□
    ■■■■■□□□□■■■■■■   ■■□□□□□□□□□□■■■   □□□□□□□□□□□□□□■   □□□□□□□□□□□□□□□
    ■■■■■■□□■■■■■■■   ■■■□□□□□□□□■■■■   ■□□□□□□□□□□□□■■   □□□□□□□□□□□□□□□
    ■■■■■■■■■■■■■■■   ■■■■□□□□□□■■■■■   ■■□□□□□□□□□□■■■   □□□□□□□□□□□□□□■

``RadialDirection::Inward`` is the mirror: a shrinking diamond of screen A
disappears into screen B instead.

Shutters
~~~~~~~~

Evenly spaced slats reveal the new screen in alternating bands::

    new ShuttersTransition(VerticalDirection::Top, size: 2);

.. code-block:: text

    □□□□□□■■■■■■■■■   □□□□□□□□□□□□□■■   □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□
    ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■   □□□□□■■■■■■■■■■   □□□□□□□□□□□□□■■
    □□□□□□■■■■■■■■■   □□□□□□□□□□□□□■■   □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□
    ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■   □□□□□■■■■■■■■■■   □□□□□□□□□□□□□■■
    □□□□□□■■■■■■■■■   □□□□□□□□□□□□□■■   □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□

Spiral
~~~~~~

The new screen unfolds block by block along a spiral path::

    new SpiralTransition(RadialDirection::Outward, size: 3);

.. code-block:: text

    ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■   ■■■■■■■■■□□□□□□   ■■■□□□□□□□□□□□□
    ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■   ■■■■■■■■■□□□□□□   ■■■□□□□□□□□□□□□
    ■■■■■■■■■■■■■■■   ■■■■■■■■■■■■■■■   ■■■■■■■■■□□□□□□   ■■■□□□□□□□□□□□□
    □□□□□□■■■■■■■■■   □□□□□□□□□□□□■■■   □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□
    □□□□□□■■■■■■■■■   □□□□□□□□□□□□■■■   □□□□□□□□□□□□□□□   □□□□□□□□□□□□□□□

Custom Transitions
------------------

Extend ``AbstractTransition`` and implement ``doBlend()``::

    use Symfony\Component\Tui\Transition\AbstractTransition;

    final class FadeToBlackTransition extends AbstractTransition
    {
        protected function doBlend(
            array $fromLines,
            array $toLines,
            int $width,
            int $height,
            float $progress,
        ): array
        {
            return $progress < 0.5 ? $fromLines : $toLines;
        }
    }

``$fromLines`` and ``$toLines`` are the two screens as ANSI-formatted line
arrays, with one string per row. ``$progress`` is already eased and clamped.
``AbstractTransition`` returns the source or destination screen before calling
``doBlend()`` when the widget's progress is exactly ``0.0`` or ``1.0``. Pass
your transition instance to ``start()`` like a built-in transition.

Styling
-------

Like any widget, ``TransitionWidget`` accepts padding, borders and a background
through the Style system (see :doc:`/tui/style/index`). The ``from`` and ``to``
widgets control what you see during a transition. Style ``TransitionWidget``
for elements around the whole stage, such as a container border.

Events
------

``TransitionWidget`` does not emit any events.
