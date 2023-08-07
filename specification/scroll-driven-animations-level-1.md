# Scroll-driven Animations
## 1. Введение
--------------------------

Данная спецификация определяет механизмы управления ходом анимации на основе хода прокрутки контейнера прокрутки. Такие анимации, управляемые прокруткой, используют временную шкалу, основанную на положении прокрутки, а не на времени. Данный модуль предоставляет как императивный API, основанный на Web Animations API, так и декларативный API, основанный на CSS Animations. [\[WEB-ANIMATIONS-1\]](#biblio-web-animations-1 "Web Animations")

Существует два типа временных шкал, управляемых прокруткой:

*   [Scroll Progress Timelines](#scroll-timelines), которые связаны с прогрессом прокрутки конкретного [scroll container](https://www.w3.org/TR/css-overflow-3/#scroll-container)
*   [View Progress Timelines](#view-timelines), которые связаны с ходом просмотра конкретного [box](https://www.w3.org/TR/css-display-3/#box) through a [scrollport](https://www.w3.org/TR/css-overflow-3/#scrollport)


> Примечание: Анимации, управляемые прокруткой, прогресс которых связан с позицией прокрутки, отличаются от анимаций, управляемых прокруткой, которые запускаются позицией прокрутки, но прогресс которых определяется временем.

### 1.1. Взаимосвязь с другими спецификациями

Web Animations [\[WEB-ANIMATIONS-1\]](#biblio-web-animations-1 "Web Animations") определяет абстрактную концептуальную модель анимации на платформе Web, элементы которой включают [анимации](https://www.w3.org/TR/web-animations-1/#concept-animation) и их [временные линии](https://www.w3.org/TR/web-animations-1/#timeline), а также соответствующие программные интерфейсы. Данная спецификация расширяет модель Web Animations, определяя [scroll-driven timelines](#scroll-driven-timelines) и позволяя управлять прогрессом в анимации для создания [scroll-driven animations](#scroll-driven-animations).

В данной спецификации представлены как интерфейсы программирования для взаимодействия с этими концепциями, так и свойства CSS, применяющие эти концепции к CSS-анимациям [\[CSS-ANIMATIONS-1\]] (#biblio-css-animations-1 "CSS Animations Level 1"). В той мере, в какой поведение этих свойств CSS описывается в терминах интерфейсов программирования, [User Agents](https://infra.spec.whatwg.org/#user-agent), не поддерживающие сценарии, могут соответствовать данной спецификации, реализуя свойства CSS таким образом, чтобы они вели себя так, как если бы имелись базовые интерфейсы программирования.

Как и большинство других операций в CSS, помимо сопоставления [селектора](https://www.w3.org/TR/CSS21/syndata.html#x15), функции в данной спецификации работают над [уплощенным деревом элементов](https://drafts.csswg.org/css-scoping-1/#flat-tree).

### 1.2. Взаимосвязь с асинхронной прокруткой

Некоторые пользовательские агенты поддерживают прокрутку, которая является асинхронной по отношению к макету или сценарию. Данная спецификация предназначена для совместимости с такой архитектурой.

В частности, данная спецификация позволяет выразить эффекты, управляемые прокруткой, таким образом, чтобы не требовать выполнения сценария при каждой выборке эффекта. Пользовательским агентам, поддерживающим асинхронную прокрутку, разрешается (но не требуется) асинхронная выборка таких эффектов.

### 1.3. Определения стоимости

Данная спецификация следует [соглашениям определения свойств CSS](https://www.w3.org/TR/CSS2/about.html#property-defs) из [\[CSS2\]](#biblio-css2 "Cascading Style Sheets Level 2 Revision 1 (CSS 2.1) Specification"), используя [синтаксис определения значений](https://www.w3.org/TR/css-values-3/#value-defs) из [\[CSS-VALUES-3\]](#biblio-css-values-3 "CSS Values and Units Module Level 3"). Типы значений, не определенные в данной спецификации, определяются в CSS Values & Units \[CSS-VALUES-3\]. Комбинация с другими модулями CSS может расширить определения этих типов значений.

В дополнение к значениям свойств, перечисленным в их определениях, все свойства, определенные в данной спецификации, также принимают в качестве значения свойства [CSS-wide keywords](https://www.w3.org/TR/css-values-4/#css-wide-keywords). Для удобства чтения они не повторяются в явном виде.

## 2. Графики выполнения прокрутки

Временные шкалы прокрутки - это шкалы, привязанные к прогрессу в положении прокрутки [контейнера прокрутки](https://www.w3.org/TR/css-overflow-3/#scroll-container) вдоль определенной оси. Самая начальная позиция прокрутки представляет собой 0% прогресса, а самая конечная - 100% прогресса.

На [Scroll-progress-timelines](#scroll-progress-timelines) можно ссылаться в [animation-timeline](https://www.w3.org/TR/css-animations-2/#propdef-animation-timeline) анонимно, используя [scroll()](#funcdef-scroll) [функциональную нотацию](https://www.w3.org/TR/css-values-4/#functional-notation) или по имени (см. [§ 4.2 Named Timeline Scoping and Lookup](#timeline-scoping)) после объявления их с помощью свойств [scroll-timeline](#propdef-scroll-timeline). В Web Animations API они могут быть представлены анонимно объектом `[ScrollTimeline](#scrolltimeline)`.

### 2.1. Расчет прогресса для временной шкалы прокрутки

Прогресс ([текущее время](https://www.w3.org/TR/web-animations-1/#timeline-current-time)) для [scroll progress timeline](#scroll-progress-timelines) вычисляется как: [scroll offset](https://www.w3.org/TR/css-overflow-3/#scroll-offset) ÷ ([scrollable overflow](https://www.w3.org/TR/css-overflow-3/#scrollable-overflow) size - [scroll container](https://www.w3.org/TR/css-overflow-3/#scroll-container) size).

Если позиция 0% и позиция 100% совпадают (т.е. знаменатель в формуле [current time](https://www.w3.org/TR/web-animations-1/#timeline-current-time) равен нулю), то временная шкала является [неактивной](https://www.w3.org/TR/web-animations-1/#inactive-timeline).

В [paged media](https://www.w3.org/TR/mediaqueries-5/#paged-media) временные шкалы [scroll progress timelines](#scroll-progress-timelines), которые в противном случае ссылались бы на область просмотра документа, также являются [неактивными](https://www.w3.org/TR/web-animations-1/#inactive-timeline).

### 2.2. Сроки выполнения анонимного прокручивания

#### 2.2.1. Нотация scroll()

Функциональная нотация scroll() может использоваться в качестве значения [<single-animation-timeline>](https://www.w3.org/TR/css-animations-2/#typedef-single-animation-timeline "Expands to: auto | none") в [animation-timeline](https://www.w3.org/TR/css-animations-2/#propdef-animation-timeline) и задавать [scroll progress timeline](#scroll-progress-timelines). Его синтаксис следующий
```
<scroll()> = scroll( [ <scroller> || <axis> ]? )
<axis> = block | inline | x | y
<scroller> = root | nearest | self

```


По умолчанию [scroll()](#funcdef-scroll) ссылается на [block axis](https://www.w3.org/TR/css-writing-modes-4/#block-axis) ближайшего предка [scroll container](https://www.w3.org/TR/css-overflow-3/#scroll-container). Его аргументы изменяют этот поиск следующим образом:

**block**

Указывает на использование меры продвижения вдоль [оси блока](https://www.w3.org/TR/css-writing-modes-4/#block-axis) [контейнера прокрутки](https://www.w3.org/TR/css-overflow-3/#scroll-container). (По умолчанию.)

**inline**

Указывает на использование меры прогресса вдоль [inline axis](https://www.w3.org/TR/css-writing-modes-4/#inline-axis) [scroll container](https://www.w3.org/TR/css-overflow-3/#scroll-container).

**x**

Указывает на использование меры прогресса вдоль [горизонтальной оси](https://www.w3.org/TR/css-writing-modes-4/#x-axis) [контейнера прокрутки](https://www.w3.org/TR/css-overflow-3/#scroll-container).

**y**

Указывает на использование меры прогресса вдоль [вертикальной оси](https://www.w3.org/TR/css-writing-modes-4/#y-axis) [контейнера прокрутки](https://www.w3.org/TR/css-overflow-3/#scroll-container).
**nearest**

Указывает на использование ближайшего предка [контейнера прокрутки](https://www.w3.org/TR/css-overflow-3/#scroll-container). (По умолчанию.)

**root**

Указывает на использование области просмотра документа в качестве [контейнера прокрутки](https://www.w3.org/TR/css-overflow-3/#scroll-container).

**self**

Указывает на использование собственного [основного блока](https://www.w3.org/TR/css-display-3/#principal-box) элемента в качестве [контейнера прокрутки](https://www.w3.org/TR/css-overflow-3/#scroll-container). Если основной блок не является контейнером прокрутки, то [временная шкала прокрутки](#scroll-progress-timelines) будет [неактивной](https://www.w3.org/TR/web-animations-1/#inactive-timeline).

> Примечание: Прогресс относится к [началу прокрутки](https://www.w3.org/TR/css-overflow-3/#scroll-origin), которое может меняться в зависимости от [режима записи](https://www.w3.org/TR/css-writing-modes-4/#writing-mode), даже если заданы [x](#valdef-scroll-x) или [y](#valdef-scroll-y).

Ссылки на [корневой элемент](https://www.w3.org/TR/css-display-3/#root-element) распространяются на область просмотра документа (которая функционирует как его [контейнер прокрутки](https://www.w3.org/TR/css-overflow-3/#scroll-container)).

Каждое использование [scroll()](#funcdef-scroll) соответствует собственному экземпляру `[ScrollTimeline](#scrolltimeline)` в Web Animations API, даже если несколько элементов используют scroll() для ссылки на один и тот же [контейнер прокрутки](https://www.w3.org/TR/css-overflow-3/#scroll-container) с одинаковыми аргументами.

#### 2.2.2. Интерфейс `ScrollTimeline`.

```
enum ScrollAxis {
  "block",
  "inline",
  "x",
  "y"
};

dictionary ScrollTimelineOptions {
  Element? source;
  ScrollAxis axis = "block";
};

[Exposed=Window]
interface ScrollTimeline : AnimationTimeline {
  constructor(optional ScrollTimelineOptions options = {});
  readonly attribute Element? source;
  readonly attribute ScrollAxis axis;
};

```


ScrollTimeline - это `[AnimationTimeline](https://www.w3.org/TR/web-animations-1/#animationtimeline)`, представляющий собой временную шкалу [прогресса прокрутки](#scroll-progress-timelines). Его можно передать в конструктор `[Animation](https://www.w3.org/TR/web-animations-1/#animation)` или в метод `[animate()](https://www.w3.org/TR/web-animations-1/#dom-animatable-animate)` для привязки анимации к временной шкале прокрутки.

Источник", типа [Element](https://dom.spec.whatwg.org/#element), readonly, nullable

Элемент [scroll container](https://www.w3.org/TR/css-overflow-3/#scroll-container), позиция прокрутки которого управляет ходом временной шкалы.

`axis`, типа [ScrollAxis](#enumdef-scrollaxis), readonly

Ось прокрутки, по которой движется временная шкала. См. определения значений для [<axis>](#typedef-axis), выше.

Наследуемые атрибуты:

`[CurrentTime](https://www.w3.org/TR/web-animations-1/#dom-animationtimeline-currenttime)` (наследуется от `[AnimationTimeline](https://www.w3.org/TR/web-animations-1/#animationtimeline)`)

Представляет прогресс прокрутки [контейнера прокрутки](https://www.w3.org/TR/css-overflow-3/#scroll-container) в виде процентного CSSUnitValue, причем 0% представляет собой крайнее начальное положение прокрутки (в [режиме записи](https://www.w3.org/TR/css-writing-modes-4/#writing-mode) контейнера прокрутки). Null, если временная шкала [неактивна](https://www.w3.org/TR/web-animations-1/#inactive-timeline).

> [](#issue-73faeaf7)Хотя 0% обычно представляет собой начальную позицию прокрутки [контейнера прокрутки](https://www.w3.org/TR/css-overflow-3/#scroll-container), это может не совпадать с ней в зависимости от [распределения содержимого](https://www.w3.org/TR/css-align-3/#content-distribute). См. [CSS Box Alignment 3 § 5.3 Overflow and Scroll Positions](https://www.w3.org/TR/css-align-3/#overflow-scroll-position). Это то, что нам нужно?

> [](#issue-00572649)Добавить примечание о том, может ли `[currentTime](https://www.w3.org/TR/web-animations-1/#dom-animationtimeline-currenttime)` быть отрицательным или > 100%.

`ScrollTimeline(options)`

Создает новый объект `[ScrollTimeline](#scrolltimeline)` с помощью следующей процедуры:

1.  Пусть timeline - новый объект `[ScrollTimeline](#scrolltimeline)`.

2.  Установите для timeline значение `[source](#dom-scrolltimeline-source)`:

    Если член `source` опции присутствует и не является null,

    Член `source` опции.

    Иначе,

    `[scrollingElement](https://www.w3.org/TR/cssom-view-1/#dom-document-scrollingelement)` из `[Document](https://dom.spec.whatwg.org/#document)` [связанного](https://html.spec.whatwg.org/multipage/window-object.html#concept-document-window) с `[Window](https://html.spec.whatwg.org/multipage/nav-history-apis.html#window)`, который является [текущим глобальным объектом](https://html.spec.whatwg.org/multipage/webappapis.html#current-global-object).

3.  Установите свойство `[axis](#dom-scrolltimeline-axis)` временной шкалы в соответствующее значение из опций.


Если `[источник](#dom-scrolltimeline-source)` элемента `[ScrollTimeline](#scrolltimeline)` является элементом, чей [основной блок](https://www.w3.org/TR/css-display-3/#principal-box) не существует или не является [контейнером прокрутки](https://www.w3.org/TR/css-overflow-3/#scroll-container), или если нет [прокручиваемого переполнения](https://www.w3.org/TR/css-overflow-3/#scrollable-overflow), то `[ScrollTimeline](#scrolltimeline)` является [неактивным](https://www.w3.org/TR/web-animations-1/#inactive-timeline).

Длительность `[ScrollTimeline](#scrolltimeline)` равна 100%.

Значения `[source](#dom-scrolltimeline-source)` и `[currentTime](https://www.w3.org/TR/web-animations-1/#dom-animationtimeline-currenttime)` вычисляются при запросе или обновлении.

### 2.3. Named Scroll Progress Timelines

[Scroll-progress-timelines] (#scroll-progress-timelines) также может быть определен на самом [scroll container] (https://www.w3.org/TR/css-overflow-3/#scroll-container), а затем по имени обращаться к элементам в области видимости этого имени (см. [§ 4.2 Named Timeline Scoping and Lookup] (#timeline-scoping)]).

Такие именованные временные шкалы прокрутки объявляются в [согласованном списке значений](https://www.w3.org/TR/css-values-4/#coordinated-value-list), построенном из [длинных](https://www.w3.org/TR/css-cascade-5/#longhand) [сокращенных](https://www.w3.org/TR/css-cascade-5/#shorthand-property) свойств [scroll-timeline](#propdef-scroll-timeline), которые образуют группу свойств [согласованного списка](https://www.w3.org/TR/css-values-4/#coordinating-list-property) с [scroll-timeline-name](#propdef-scroll-timeline-name) в качестве базового свойства [согласованного списка](https://www.w3.org/TR/css-values-4/#coordinating-list-base-property). См. [CSS Values 4 § A Coordinating List-Valued Properties](https://www.w3.org/TR/css-values-4/#linked-properties).

#### 2.3.1. Именование временной шкалы прокрутки: свойство [scroll-timeline-name](#propdef-scroll-timeline-name)


|Name:                 |scroll-timeline-name                               |
|----------------------|---------------------------------------------------|
|Value:                |none | <dashed-ident>#                             |
|Initial:              |none                                               |
|Applies to:           |all elements                                       |
|Inherited:            |no                                                 |
|Percentages:          |n/a                                                |
|Computed value:       |the keyword none or a list of CSS identifiers      |
|Canonical order:      |per grammar                                        |
|Animation type:       |not animatable                                     |


Определяет имена для [именованных временных шкал прокрутки](#named-scroll-progress-timelines), связанных с этим элементом.

#### 2.3.2. Ось временной шкалы прокрутки: свойство [scroll-timeline-axis](#propdef-scroll-timeline-axis)


|Name:                 |scroll-timeline-axis                  |
|----------------------|--------------------------------------|
|Value:                |[ block | inline | x | y ]#           |
|Initial:              |block                                 |
|Applies to:           |all elements                          |
|Inherited:            |no                                    |
|Percentages:          |n/a                                   |
|Computed value:       |a list of the keywords specified      |
|Canonical order:      |per grammar                           |
|Animation type:       |not animatable                        |


Определяет ось любой [именованной временной шкалы прокрутки](#named-scroll-progress-timelines), исходящей из данного [контейнера прокрутки](https://www.w3.org/TR/css-overflow-3/#scroll-container). Если этот бокс не является контейнером прокрутки, то соответствующая именованная временная шкала прокрутки будет [неактивной](https://www.w3.org/TR/web-animations-1/#inactive-timeline).

Значения соответствуют определению для [scroll()](#funcdef-scroll).

#### 2.3.3. Сокращение временной шкалы прокрутки: сокращение [scroll-timeline](#propdef-scroll-timeline)


|Name:                 |scroll-timeline                                              |
|----------------------|-------------------------------------------------------------|
|Value:                |[ <'scroll-timeline-name'> <'scroll-timeline-axis'>? ]#      |
|Initial:              |see individual properties                                    |
|Applies to:           |all elements                                                 |
|Inherited:            |no                                                           |
|Percentages:          |see individual properties                                    |
|Computed value:       |see individual properties                                    |
|Animation type:       |not animatable                                               |
|Canonical order:      |per grammar                                                  |


Это свойство является [сокращением](https://www.w3.org/TR/css-cascade-5/#shorthand-property) для задания [scroll-timeline-name](#propdef-scroll-timeline-name) и [scroll-timeline-axis](#propdef-scroll-timeline-axis) в одном объявлении.

## 3. View Progress Timelines
----------------------------------------------

Often animations are desired to start and end during the portion of the [scroll progress timeline](#scroll-progress-timelines) that a particular box (the view progress subject) is in view within the [scrollport](https://www.w3.org/TR/css-overflow-3/#scrollport). View progress timelines are segments of a scroll progress timeline that are scoped to the scroll positions in which any part of the subject element’s [principal box](https://www.w3.org/TR/css-display-3/#principal-box) intersects its nearest ancestor scrollport (or more precisely, the relevant [view progress visibility range](#view-progress-visibility-range) of that scrollport). The startmost such scroll position represents 0% progress, and the endmost such scroll position represents 100% progress; see [§ 3.2 Calculating Progress for a View Progress Timeline](#view-timeline-progress).

Note: The 0% and 100% scroll positions are not always reachable, e.g. if the box is positioned at the start edge of the [scrollable overflow rectangle](https://www.w3.org/TR/css-overflow-3/#scrollable-overflow-rectangle), it might not be possible to scroll to < 32% progress.

[View progress timelines](#view-progress-timelines) can be referenced anonymously using the [view()](#funcdef-view) [functional notation](https://www.w3.org/TR/css-values-4/#functional-notation) or by name (see [§ 4.2 Named Timeline Scoping and Lookup](#timeline-scoping)) after declaring them using the [view-timeline](#propdef-view-timeline) properties on the [view progress subject](#view-progress-subject). In the Web Animations API, they can be represented anonymously by a `[ViewTimeline](#viewtimeline)` object.

### 3.1. View Progress Timeline Ranges[](#view-timelines-ranges)

[View progress timelines](#view-progress-timelines) define the following [named timeline ranges](#named-timeline-range):

cover

Represents the full range of the [view progress timeline](#view-progress-timelines):

*   0% progress represents the latest position at which the [start](https://dom.spec.whatwg.org/#concept-range-start) [border edge](https://www.w3.org/TR/css-box-4/#border-edge) of the element’s [principal box](https://www.w3.org/TR/css-display-3/#principal-box) coincides with the [end](https://dom.spec.whatwg.org/#concept-range-end) edge of its [view progress visibility range](#view-progress-visibility-range).

*   100% progress represents the earliest position at which the [end](https://dom.spec.whatwg.org/#concept-range-end) [border edge](https://www.w3.org/TR/css-box-4/#border-edge) of the element’s [principal box](https://www.w3.org/TR/css-display-3/#principal-box) coincides with the [start](https://dom.spec.whatwg.org/#concept-range-start) edge of its [view progress visibility range](#view-progress-visibility-range).


contain

Represents the range during which the [principal box](https://www.w3.org/TR/css-display-3/#principal-box) is either fully contained by, or fully covers, its [view progress visibility range](#view-progress-visibility-range) within the [scrollport](https://www.w3.org/TR/css-overflow-3/#scrollport).

*   0% progress represents the earliest position at which either:

    *   the [start](https://dom.spec.whatwg.org/#concept-range-start) [border edge](https://www.w3.org/TR/css-box-4/#border-edge) of the element’s [principal box](https://www.w3.org/TR/css-display-3/#principal-box) coincides with the start edge of its [view progress visibility range](#view-progress-visibility-range).

    *   the [end](https://dom.spec.whatwg.org/#concept-range-end) [border edge](https://www.w3.org/TR/css-box-4/#border-edge) of the element’s [principal box](https://www.w3.org/TR/css-display-3/#principal-box) coincides with the end edge of its [view progress visibility range](#view-progress-visibility-range).

*   100% progress represents the latest position at which either:

    *   the [start](https://dom.spec.whatwg.org/#concept-range-start) [border edge](https://www.w3.org/TR/css-box-4/#border-edge) of the element’s [principal box](https://www.w3.org/TR/css-display-3/#principal-box) coincides with the start edge of its [view progress visibility range](#view-progress-visibility-range).

    *   the [end](https://dom.spec.whatwg.org/#concept-range-end) [border edge](https://www.w3.org/TR/css-box-4/#border-edge) of the element’s [principal box](https://www.w3.org/TR/css-display-3/#principal-box) coincides with the end edge of its [view progress visibility range](#view-progress-visibility-range).


entry[](#valdef-animation-timeline-range-entry)

Represents the range during which the [principal box](https://www.w3.org/TR/css-display-3/#principal-box) is entering the [view progress visibility range](#view-progress-visibility-range).

*   0% is equivalent to 0% of the [cover](#valdef-animation-timeline-range-cover) range.

*   100% is equivalent to 0% of the [contain](#valdef-animation-timeline-range-contain) range.


exit[](#valdef-animation-timeline-range-exit)

Represents the range during which the [principal box](https://www.w3.org/TR/css-display-3/#principal-box) is exiting the [view progress visibility range](#view-progress-visibility-range).

*   0% is equivalent to 100% of the [contain](#valdef-animation-timeline-range-contain) range.

*   100% is equivalent to 100% of the [cover](#valdef-animation-timeline-range-cover) range.


entry-crossing[](#valdef-animation-timeline-range-entry-crossing)

Represents the range during which the [principal box](https://www.w3.org/TR/css-display-3/#principal-box) crosses the [end](https://dom.spec.whatwg.org/#concept-range-end) [border edge](https://www.w3.org/TR/css-box-4/#border-edge)

*   0% progress represents the latest position at which the [start](https://dom.spec.whatwg.org/#concept-range-start) [border edge](https://www.w3.org/TR/css-box-4/#border-edge) of the element’s [principal box](https://www.w3.org/TR/css-display-3/#principal-box) coincides with the [end](https://dom.spec.whatwg.org/#concept-range-end) edge of its [view progress visibility range](#view-progress-visibility-range).

*   100% progress represents the earliest position at which the [end](https://dom.spec.whatwg.org/#concept-range-end) [border edge](https://www.w3.org/TR/css-box-4/#border-edge) of the element’s [principal box](https://www.w3.org/TR/css-display-3/#principal-box) coincides with the end edge of its [view progress visibility range](#view-progress-visibility-range).


exit-crossing[](#valdef-animation-timeline-range-exit-crossing)

Represents the range during which the [principal box](https://www.w3.org/TR/css-display-3/#principal-box) crosses the [start](https://dom.spec.whatwg.org/#concept-range-start) [border edge](https://www.w3.org/TR/css-box-4/#border-edge)

*   0% progress represents the latest position at which the [start](https://dom.spec.whatwg.org/#concept-range-start) [border edge](https://www.w3.org/TR/css-box-4/#border-edge) of the element’s [principal box](https://www.w3.org/TR/css-display-3/#principal-box) coincides with the start edge of its [view progress visibility range](#view-progress-visibility-range).

*   100% progress represents the earliest position at which the [end](https://dom.spec.whatwg.org/#concept-range-end) [border edge](https://www.w3.org/TR/css-box-4/#border-edge) of the element’s [principal box](https://www.w3.org/TR/css-display-3/#principal-box) coincides with the [start](https://dom.spec.whatwg.org/#concept-range-start) edge of its [view progress visibility range](#view-progress-visibility-range).


[](#issue-aab79dad)Insert diagrams.

In all cases, the [writing mode](https://www.w3.org/TR/css-writing-modes-4/#writing-mode) used to resolve the [start](https://dom.spec.whatwg.org/#concept-range-start) and [end](https://dom.spec.whatwg.org/#concept-range-end) sides is the writing mode of the relevant [scroll container](https://www.w3.org/TR/css-overflow-3/#scroll-container). [Transforms](https://www.w3.org/TR/css-transforms/) are ignored, but [relative](https://www.w3.org/TR/CSS21/visuren.html#x34) and [absolute](https://www.w3.org/TR/css-position-3/#absolute-position) positioning are accounted for.

Note: For [sticky-positioned boxes](https://www.w3.org/TR/css-position-3/#sticky-position) the 0% and 100% progress conditions can sometimes be satisfied by a range of scroll positions rather than just one. Each range therefore indicates whether to use the earliest or latest qualifying position.

[\[CSS-POSITION-3\]](#biblio-css-position-3 "CSS Positioned Layout Module Level 3") [\[CSS-TRANSFORMS-1\]](#biblio-css-transforms-1 "CSS Transforms Module Level 1")

### 3.2. Calculating Progress for a View Progress Timeline[](#view-timeline-progress)

Progress (the [current time](https://www.w3.org/TR/web-animations-1/#timeline-current-time)) in a [view progress timeline](#view-progress-timelines) is calculated as: distance ÷ range where:

*   distance is the current [scroll offset](https://www.w3.org/TR/css-overflow-3/#scroll-offset) minus the scroll offset corresponding to the start of the [cover](#valdef-animation-timeline-range-cover) range

*   range is the [scroll offset](https://www.w3.org/TR/css-overflow-3/#scroll-offset) corresponding to the start of the [cover](#valdef-animation-timeline-range-cover) range minus the scroll offset corresponding to the end of the cover range


If the 0% position and 100% position coincide (i.e. the denominator in the [current time](https://www.w3.org/TR/web-animations-1/#timeline-current-time) formula is zero), the timeline is [inactive](https://www.w3.org/TR/web-animations-1/#inactive-timeline).

In [paged media](https://www.w3.org/TR/mediaqueries-5/#paged-media), [view progress timelines](#view-progress-timelines) that would otherwise reference the document viewport are also [inactive](https://www.w3.org/TR/web-animations-1/#inactive-timeline).

### 3.3. Anonymous View Progress Timelines[](#view-timelines-anonymous)

#### 3.3.1. The [view()](#funcdef-view) notation[](#view-notation)

The view() functional notation can be used as a [<single-animation-timeline>](https://www.w3.org/TR/css-animations-2/#typedef-single-animation-timeline "Expands to: auto | none") value in [animation-timeline](https://www.w3.org/TR/css-animations-2/#propdef-animation-timeline) and specifies a [view progress timeline](#view-progress-timelines) in reference to the nearest ancestor [scroll container](https://www.w3.org/TR/css-overflow-3/#scroll-container). Its syntax is

```
<view()> = view( [ <axis> || <'view-timeline-inset'> ]? )

```


By default, [view()](#funcdef-view) references the [block axis](https://www.w3.org/TR/css-writing-modes-4/#block-axis); as for [scroll()](#funcdef-scroll), this can be changed by providing an explicit [<axis>](#typedef-axis) value.

The optional [<'view-timeline-inset'>](#propdef-view-timeline-inset) value provides an adjustment of the [view progress visibility range](#view-progress-visibility-range), as defined for view-timeline-inset.

Each use of [view()](#funcdef-view) corresponds to its own instance of `[ViewTimeline](#viewtimeline)` in the Web Animations API, even if multiple elements use view() to reference the same element with the same arguments.

#### 3.3.2. The `[ViewTimeline](#viewtimeline)` Interface[](#viewtimeline-interface)

```
dictionary ViewTimelineOptions {
  Element subject;
  ScrollAxis axis = "block";
  (DOMString or sequence<(CSSNumericValue or CSSKeywordValue)>) ) " id="dom-viewtimelineoptions-inset">inset = "auto";
};

[Exposed=Window]
interface ViewTimeline : ScrollTimeline {
  constructor(optional ViewTimelineOptions options = {});
  readonly attribute Element subject;
  readonly attribute CSSNumericValue startOffset;
  readonly attribute CSSNumericValue endOffset;
};

```


A `[ViewTimeline](#viewtimeline)` is an `[AnimationTimeline](https://www.w3.org/TR/web-animations-1/#animationtimeline)` that specifies a [view progress timeline](#view-progress-timelines). It can be passed to the `[Animation](https://www.w3.org/TR/web-animations-1/#animation)` constructor or the `[animate()](https://www.w3.org/TR/web-animations-1/#dom-animatable-animate)` method to link the animation to a view progress timeline.

`subject`, of type [Element](https://dom.spec.whatwg.org/#element), readonly

The element whose [principal box](https://www.w3.org/TR/css-display-3/#principal-box)’s visibility in the [scrollport](https://www.w3.org/TR/css-overflow-3/#scrollport) defines the progress of the timeline.

`startOffset`, of type [CSSNumericValue](https://www.w3.org/TR/css-typed-om-1/#cssnumericvalue), readonly

Represents the starting (0% progress) scroll position of the [view progress timeline](#view-progress-timelines) as a length offset (in [px](https://www.w3.org/TR/css-values-4/#px)) from the [scroll origin](https://www.w3.org/TR/css-overflow-3/#scroll-origin). Null when the timeline is [inactive](https://www.w3.org/TR/web-animations-1/#inactive-timeline).

`endOffset`, of type [CSSNumericValue](https://www.w3.org/TR/css-typed-om-1/#cssnumericvalue), readonly

Represents the ending (100% progress) scroll position of the [view progress timeline](#view-progress-timelines) as a length offset (in [px](https://www.w3.org/TR/css-values-4/#px)) from the [scroll origin](https://www.w3.org/TR/css-overflow-3/#scroll-origin). Null when the timeline is [inactive](https://www.w3.org/TR/web-animations-1/#inactive-timeline).

Note: The values of `[startOffset](#dom-viewtimeline-startoffset)` and `[endOffset](#dom-viewtimeline-endoffset)` are relative to the [scroll origin](https://www.w3.org/TR/css-overflow-3/#scroll-origin), not the [physical](https://www.w3.org/TR/css-writing-modes-4/#physical) top left corner. Depending on the [writing mode](https://www.w3.org/TR/css-writing-modes-4/#writing-mode) of the [scroll container](https://www.w3.org/TR/css-overflow-3/#scroll-container), they therefore might not match `[scrollLeft](https://www.w3.org/TR/cssom-view-1/#dom-element-scrollleft)` or `[scrollTop](https://www.w3.org/TR/cssom-view-1/#dom-element-scrolltop)` values, for example in the [horizontal axis](https://www.w3.org/TR/css-writing-modes-4/#x-axis) in a right-to-left ([rtl](https://www.w3.org/TR/css-writing-modes-4/#valdef-direction-rtl)) writing mode.

Inherited attributes:

`[source](#dom-scrolltimeline-source)` (inherited from `[ScrollTimeline](#scrolltimeline)`)

The nearest ancestor of the `[subject](#dom-viewtimeline-subject)` whose [principal box](https://www.w3.org/TR/css-display-3/#principal-box) establishes a [scroll container](https://www.w3.org/TR/css-overflow-3/#scroll-container), whose scroll position drives the progress of the timeline.

`[axis](#dom-scrolltimeline-axis)` (inherited from `[ScrollTimeline](#scrolltimeline)`)

Specifies the axis of scrolling that drives the progress of the timeline. See [<axis>](#typedef-axis), above.

`[currentTime](https://www.w3.org/TR/web-animations-1/#dom-animationtimeline-currenttime)` (inherited from `[AnimationTimeline](https://www.w3.org/TR/web-animations-1/#animationtimeline)`)

Represents the current progress of the [view progress timeline](#view-progress-timelines) as a percentage `[CSSUnitValue](https://www.w3.org/TR/css-typed-om-1/#cssunitvalue)` representing its [scroll container](https://www.w3.org/TR/css-overflow-3/#scroll-container)’s scroll progress at that position. Null when the timeline is [inactive](https://www.w3.org/TR/web-animations-1/#inactive-timeline).

`ViewTimeline(options)`

Creates a new `[ViewTimeline](#viewtimeline)` object using the following procedure:

1.  Let timeline be the new `[ViewTimeline](#viewtimeline)` object.

2.  Set the `[subject](#dom-viewtimeline-subject)` and `[axis](#dom-scrolltimeline-axis)` properties of timeline to the corresponding values from options.

3.  Set the `[source](#dom-scrolltimeline-source)` of timeline to the `[subject](#dom-viewtimeline-subject)`’s nearest ancestor [scroll container](https://www.w3.org/TR/css-overflow-3/#scroll-container) element.

4.  If a `[DOMString](https://webidl.spec.whatwg.org/#idl-DOMString)` value is provided as an inset, parse it as a [<'view-timeline-inset'>](#propdef-view-timeline-inset) value; if a sequence is provided, the first value represents the start inset and the second value represents the end inset. If the sequence has only one value, it is duplicated. If it has zero values or more than two values, or if it contains a `[CSSKeywordValue](https://www.w3.org/TR/css-typed-om-1/#csskeywordvalue)` whose `[value](https://www.w3.org/TR/css-typed-om-1/#dom-csskeywordvalue-value)` is not "auto", throw a TypeError.

    These insets define the `[ViewTimeline](#viewtimeline)`’s [view progress visibility range](#view-progress-visibility-range).


If the `[source](#dom-scrolltimeline-source)` or `[subject](#dom-viewtimeline-subject)` of a `[ViewTimeline](#viewtimeline)` is an element whose [principal box](https://www.w3.org/TR/css-display-3/#principal-box) does not exist, or if its nearest ancestor [scroll container](https://www.w3.org/TR/css-overflow-3/#scroll-container) has no [scrollable overflow](https://www.w3.org/TR/css-overflow-3/#scrollable-overflow) (or if there is no such ancestor, e.g. in print media), then the `[ViewTimeline](#viewtimeline)` is [inactive](https://www.w3.org/TR/web-animations-1/#inactive-timeline).

The values of `[subject](#dom-viewtimeline-subject)`, `[source](#dom-scrolltimeline-source)`, and `[currentTime](https://www.w3.org/TR/web-animations-1/#dom-animationtimeline-currenttime)` are all computed when any of them is requested or updated.

### 3.4. Named View Progress Timelines[](#view-timelines-named)

[View progress timelines](#view-progress-timelines) can also be defined declaratively and then referenced by name by elements within the name’s scope (see [§ 4.2 Named Timeline Scoping and Lookup](#timeline-scoping)).

Such named view progress timelines are declared in the [coordinated value list](https://www.w3.org/TR/css-values-4/#coordinated-value-list) constructed from the view-timeline-\* properties, which form a [coordinating list property group](https://www.w3.org/TR/css-values-4/#coordinating-list-property) with [view-timeline-name](#propdef-view-timeline-name) as the [coordinating list base property](https://www.w3.org/TR/css-values-4/#coordinating-list-base-property). See [CSS Values 4 § A Coordinating List-Valued Properties](https://www.w3.org/TR/css-values-4/#linked-properties).

#### 3.4.1. Naming a View Progress Timeline: the [view-timeline-name](#propdef-view-timeline-name) property[](#view-timeline-name)


|Name:                 |view-timeline-name                                 |
|----------------------|---------------------------------------------------|
|Value:                |none | <dashed-ident>#                             |
|Initial:              |none                                               |
|Applies to:           |all elements                                       |
|Inherited:            |no                                                 |
|Percentages:          |n/a                                                |
|Computed value:       |the keyword none or a list of CSS identifiers      |
|Canonical order:      |per grammar                                        |
|Animation type:       |not animatable                                     |


Specifies names for the [named view progress timelines](#named-view-progress-timelines) associated with this element.

#### 3.4.2. Axis of a View Progress Timeline: the [view-timeline-axis](#propdef-view-timeline-axis) property[](#view-timeline-axis)


|Name:                 |view-timeline-axis                    |
|----------------------|--------------------------------------|
|Value:                |[ block | inline | x | y ]#           |
|Initial:              |block                                 |
|Applies to:           |all elements                          |
|Inherited:            |no                                    |
|Percentages:          |n/a                                   |
|Computed value:       |a list of the keywords specified      |
|Canonical order:      |per grammar                           |
|Animation type:       |not animatable                        |


Specifies the axis of any [named view progress timelines](#named-view-progress-timelines) derived from this element’s [principal box](https://www.w3.org/TR/css-display-3/#principal-box).

Values are as defined for [view()](#funcdef-view).

#### 3.4.3. Inset of a View Progress Timeline: the [view-timeline-inset](#propdef-view-timeline-inset) property[](#view-timeline-inset)



* Name:      : Value:
    * view-timeline-inset     : [ [ auto | <length-percentage> ]{1,2} ]#
* Name:      : Initial:
    * view-timeline-inset     : auto
* Name:      : Applies to:
    * view-timeline-inset     : all elements
* Name:      : Inherited:
    * view-timeline-inset     : no
* Name:      : Percentages:
    * view-timeline-inset     : relative to the corresponding dimension of the relevant scrollport
* Name:      : Computed value:
    * view-timeline-inset     : a list consisting of two-value pairs representing the start and end insets each as either the keyword auto or a computed <length-percentage> value
* Name:      : Canonical order:
    * view-timeline-inset     : per grammar
* Name:      : Animation type:
    * view-timeline-inset     : by computed value type


Specifies an inset (positive) or outset (negative) adjustment of the [scrollport](https://www.w3.org/TR/css-overflow-3/#scrollport) when determining whether the box is in view when setting the bounds of the corresponding [view progress timeline](#view-progress-timelines). The first value represents the [start](https://dom.spec.whatwg.org/#concept-range-start) inset in the relevant axis; the second value represents the [end](https://dom.spec.whatwg.org/#concept-range-end) inset. If the second value is omitted, it is set to the first. The resulting range of the scrollport is the view progress visibility range.

auto

Indicates to use the value of [scroll-padding](https://www.w3.org/TR/css-scroll-snap-1/#propdef-scroll-padding).

[<length-percentage>](https://www.w3.org/TR/css-values-4/#typedef-length-percentage)[](#valdef-view-timeline-inset-length-percentage)

Like [scroll-padding](https://www.w3.org/TR/css-scroll-snap-1/#propdef-scroll-padding), defines an inward offset from the corresponding edge of the scrollport.

#### 3.4.4. View Timeline Shorthand: the [view-timeline](#propdef-view-timeline) shorthand[](#view-timeline-shorthand)


|Name:                 |view-timeline                                            |
|----------------------|---------------------------------------------------------|
|Value:                |[ <'view-timeline-name'> <'view-timeline-axis'>? ]#      |
|Initial:              |see individual properties                                |
|Applies to:           |all elements                                             |
|Inherited:            |see individual properties                                |
|Percentages:          |see individual properties                                |
|Computed value:       |see individual properties                                |
|Animation type:       |see individual properties                                |
|Canonical order:      |per grammar                                              |


This property is a [shorthand](https://www.w3.org/TR/css-cascade-5/#shorthand-property) for setting [view-timeline-name](#propdef-view-timeline-name) and [view-timeline-axis](#propdef-view-timeline-axis) in a single declaration. It does not set [view-timeline-inset](#propdef-view-timeline-inset).

[](#issue-6b6d7174)Should it reset [view-timeline-inset](#propdef-view-timeline-inset) also?

Animations can be attached to [scroll-driven timelines](#scroll-driven-timelines) using the [scroll-timeline](#propdef-scroll-timeline) property (in CSS) or the `[AnimationTimeline](https://www.w3.org/TR/web-animations-1/#animationtimeline)` parameters (in the Web Animations API). The timeline range to which their [active interval](https://www.w3.org/TR/web-animations-1/#active-interval) is attached can also be further restricted to a particular timeline range (see [Attaching Animations to Timeline Ranges](#named-range-animation-declaration)).

Time-based delays ([animation-delay](https://www.w3.org/TR/css-animations-1/#propdef-animation-delay)) do not apply to [scroll-driven animations](#scroll-driven-animations), which are distance-based.

### 4.1. Finite Timeline Calculations[](#finite-attachment)

Unlike time-driven timelines, [scroll-driven timelines](#scroll-driven-timelines) are finite, thus [scroll-driven animations](#scroll-driven-animations) are always attached to a finite [attachment range](#animation-attachment-range)—which may be further limited by [animation-range](#propdef-animation-range) (see [Appendix A: Timeline Ranges](#timeline-ranges)). The animation’s

iterations ([animation-iteration-count](https://www.w3.org/TR/css-animations-1/#propdef-animation-iteration-count)) are set within the limits of this finite range. If the specified duration is [auto](https://drafts.csswg.org/css-animations-2/#valdef-animation-duration-auto), then the remaining range is divided by its [iteration count](https://www.w3.org/TR/web-animations-1/#iteration-count) (animation-iteration-count) to find the [used](https://www.w3.org/TR/css-cascade-5/#used-value) duration.

Note: If the animation has an infinite [iteration count](https://www.w3.org/TR/web-animations-1/#iteration-count), each [iteration duration](https://www.w3.org/TR/web-animations-1/#iteration-duration)—and the resulting [active duration](https://www.w3.org/TR/web-animations-1/#active-duration)—will be zero.

Animations that include absolutely-positioned keyframes (those pinned to a specific point on the timeline, e.g. using [named timeline range keyframe selectors](#named-range-keyframes) in [@keyframes](https://www.w3.org/TR/css-animations-1/#at-ruledef-keyframes)) are assumed to have an [iteration count](https://www.w3.org/TR/web-animations-1/#iteration-count) of 1 for the purpose of finding those keyframes’ positions relative to 0% and 100%; the entire animation is then scaled to fit the [iteration duration](https://www.w3.org/TR/web-animations-1/#iteration-duration) and repeated for each iteration.

Note: It’s unclear what the use case might be for combining absolutely-positioned keyframes with iteration counts above 1; this at least gives a defined behavior. (An alternative, but perhaps weirder, behavior would be to take such absolutely-positioned keyframes “out of flow” while iterating the remaining keyframes.) The editors would be interested in hearing about any real use cases for multiple iterations here.

### 4.2. Named Timeline Scoping and Lookup[](#timeline-scoping)

A named [scroll progress timeline](#scroll-progress-timelines) or [view progress timeline](#view-progress-timelines) is referenceable by:

*   the name-declaring element itself

*   that element’s descendants


Note: The [timeline-scope](#propdef-timeline-scope) property can be used to declare the name of a timeline on an ancestor of its defining element, effectively expanding its scope beyond that element’s subtree.

If multiple elements have declared the same timeline name, the matching timeline is the one declared on the nearest element in tree order. In case of a name conflict on the same element, names declared later in the naming property ([scroll-timeline-name](#propdef-scroll-timeline-name), [view-timeline-name](#propdef-view-timeline-name)) take precedence, and [scroll progress timelines](#scroll-progress-timelines) take precedence over [view progress timelines](#view-progress-timelines).

[](#example-2a5405b2)Using timeline-scope, an element can refer to timelines bound to elements that are siblings, cousins, or even descendants. For example, the following creates an animation on an element that is linked to a [scroll progress timeline](#scroll-progress-timelines) defined by the subsequent sibling.

```
<style>
  @keyframes anim {
    from { color: red; }
    to { color: green; }
  }

  .root {
    /* declares the scope of a 'scroller' timeline to reach all descendants */
    timeline-scope: scroller;
  }

  .root .animation {
    animation: anim;
    /* references the 'scroller' timeline for driving the progress of 'anim' */
    animation-timeline: scroller;
  }

  .root .animation + .scroller {
    /* attaches a scroll progress timeline to the timeline name 'scroller' */
    scroll-timeline: scroller;
  }
</style>
&hellip;
<section class="root">
  <div class="animation">Animating Box</div>
  <div class="scroller">Scrollable Box</div>
</section>

```


### 4.3. Animation Events[](#events)

[Scroll-driven animations](#scroll-driven-animations) dispatch all the same animation events as the more typical time-driven animations as described in [Web Animations § 4.4.18 Animation events](https://www.w3.org/TR/web-animations-1/#animation-events-section), [CSS Animations 1 § 4 Animation Events](https://www.w3.org/TR/css-animations-1/#events), and [CSS Animations 2 § 4.1 Event dispatch](https://www.w3.org/TR/css-animations-2/#event-dispatch).

Note: When scrolling backwards, the `animationstart` event will fire at the _end_ of the [active interval](https://www.w3.org/TR/web-animations-1/#active-interval), and the `animationend` event will fire at the start of the active interval. However, since the `finish` event is about entering the [finished play state](https://www.w3.org/TR/web-animations-1/#finished-play-state), it only fires when scrolling forwards.

5\. Frame Calculation Details[](#frames)
----------------------------------------

### 5.1. HTML Processing Model: Event loop[](#html-processing-model-event-loop)

The ability for scrolling to drive the progress of an animation, gives rise to the possibility of layout cycles, where a change to a scroll offset causes an animation’s effect to update, which in turn causes a new change to the scroll offset.

To avoid such [layout cycles](#layout-cycles), animations with a [scroll progress timeline](#scroll-progress-timelines) update their current time once during step 7.10 of the [HTML Processing Model](https://html.spec.whatwg.org/multipage/webappapis.html#processing-model-8) event loop, as step 1 of [update animations and send events](https://www.w3.org/TR/web-animations-1/#update-animations-and-send-events).

During step 7.14.1 of the [HTML Processing Model](https://html.spec.whatwg.org/multipage/webappapis.html#processing-model-8), any created [scroll progress timelines](#scroll-progress-timelines) or [view progress timelines](#view-progress-timelines) are collected into a stale timelines set. After step 7.14 if any timelines' [named timeline ranges](#named-timeline-range) have changed, these timelines are added to the [stale timelines](#stale-timelines) set. If there are any stale timelines, they now update their current time and associated ranges, the set of stale timelines is cleared and we run and we run an additional step to recalculate styles and update layout.

Note: We check for layout changes after dispatching any `[ResizeObserver](https://www.w3.org/TR/resize-observer-1/#resizeobserver)`s intentionally to take programmatically sized elements into account.

Note: As we only gather stale timelines during the first style and layout calculation, this can only directly cause one additional style recalculation. Other APIs which require another update should be checked in the same step and be updated at the same time.

Note: Without this additional round of style and layout, [initially stale](https://www.w3.org/TR/scroll-animations-1/#initially-stale) timelines would remain stale (i.e. they would not have a current time) for the remainder of the frame where the timeline was created. This means that animations linked to such a timeline would not produce any effect value for that frame, which could lead to an undesirable initial "flash" in the rendered output.

Note: This section has no effect on forced style and layout calculations triggered by `[getComputedStyle()](https://www.w3.org/TR/cssom-1/#dom-window-getcomputedstyle)` or similar. In other words, [initially stale](https://www.w3.org/TR/scroll-animations-1/#initially-stale) timelines are visible as such through those APIs.

If the final style and layout update would result in a change in the time or scope (see [timeline-scope](#propdef-timeline-scope)) of any [scroll progress timelines](#scroll-progress-timelines) or [view progress timelines](#view-progress-timelines), they will not be re-sampled to reflect the new state until the next update of the rendering.

Nothing in this section is intended to require that scrolling block on layout or script. If a user agent normally composites frames where scrolling has occurred but the consequences of scrolling have not been fully propagated in layout or script (for example, `scroll` event listeners have not yet run), the user agent may likewise choose not to sample scroll-driven animations for that composited frame. In such cases, the rendered scroll offset and the state of a scroll-driven animation may be inconsistent in the composited frame.

6\. Privacy Considerations[](#privacy-considerations)
-----------------------------------------------------

There are no known privacy impacts of the features in this specification.

7\. Security Considerations[](#security-considerations)
-------------------------------------------------------

There are no known security impacts of the features in this specification.

Appendix A: Timeline Ranges[](#timeline-ranges)
-----------------------------------------------

[](#issue-79ed79d9)This section should move to CSS-ANIMATIONS-2 and WEB-ANIMATIONS-2.

This appendix introduces the concepts of [named timeline ranges](#named-timeline-range) and [animation attachment ranges](#animation-attachment-range) to [CSS Animations](https://www.w3.org/TR/css-animations/) and [Web Animations](https://www.w3.org/TR/web-animations/).

### Named Timeline Ranges[](#named-ranges)

A named timeline range is a named segment of an animation [timeline](https://www.w3.org/TR/web-animations-1/#timeline). The start of the segment is represented as 0% progress through the range; the end of the segment is represented as 100% progress through the range. Multiple [named timeline ranges](#named-timeline-range) can be associated with a given timeline, and multiple such ranges can overlap. For example, the contain range of a [view progress timeline](#view-progress-timelines) overlaps with its cover range. Named timeline ranges are represented by the [<timeline-range-name>](#typedef-timeline-range-name) value type, which indicates a [CSS identifier](https://www.w3.org/TR/css-values-4/#css-css-identifier) representing one of the predefined named timeline ranges.

Note: In this specification, [named timeline ranges](#named-timeline-range) must be defined to exist by a specification such as [\[SCROLL-ANIMATIONS-1\]](#biblio-scroll-animations-1 "Scroll-driven Animations"). A future level may introduce APIs for authors to declare their own custom named timeline ranges.

### Named Timeline Range Keyframe Selectors[](#named-range-keyframes)

[Named timeline range](#named-timeline-range) names and percentages can be used to attach keyframes to specific progress points within the named timeline range. The CSS [@keyframes](https://www.w3.org/TR/css-animations-1/#at-ruledef-keyframes) rule is extended thus:

```
<keyframe-selector> = from | to | <percentage [0,100]> | <timeline-range-name> <percentage>

```


where [<timeline-range-name>](#typedef-timeline-range-name) is the [CSS identifier](https://www.w3.org/TR/css-values-4/#css-css-identifier) that represents a chosen predefined [named timeline range](#named-timeline-range), and the [<percentage>](https://www.w3.org/TR/css-values-4/#percentage-value) after it represents the percentage progress between the start and end of that named timeline range.

Keyframes are attached to the specified point in the timeline. If the timeline does not have a corresponding [named timeline range](#named-timeline-range), then any keyframes attached to points on that named timeline range are ignored. It is possible that these attachment points are outside the [active interval](https://www.w3.org/TR/web-animations-1/#active-interval) of the animation; in these cases the automatic from (0%) and [to](https://drafts.csswg.org/css-shapes-2/#valdef-shape-to) (100%) keyframes are only generated for properties that don’t have keyframes at or earlier than 0% or at or after 100% (respectively).

### Attaching Animations to Timeline Ranges[](#named-range-animation-declaration)

A set of animation keyframes can be attached in reference to an animation attachment range, restricting the animation’s [active interval](https://www.w3.org/TR/web-animations-1/#active-interval) to that range of a timeline, with the [animation-range](#propdef-animation-range) properties. Delays (see [animation-delay](https://www.w3.org/TR/css-animations-1/#propdef-animation-delay)) are set within this restricted range, further reducing the time available for [auto](https://drafts.csswg.org/css-animations-2/#valdef-animation-duration-auto) durations and [infinite](https://www.w3.org/TR/css-animations-1/#valdef-animation-iteration-count-infinite) iterations.

Note: [animation-range](#propdef-animation-range) can expand the [attachment range](#animation-attachment-range) as well as constrict it.

Any frames positioned outside the [attachment range](#animation-attachment-range) are used for interpolation as needed, but are outside the [active interval](https://www.w3.org/TR/web-animations-1/#active-interval) and therefore dropped from the animation itself, effectively truncating the animation at the end of its attachment range.

```
range start┐             ╺┉┉active interval┉┉╸           ┌range end
┄┄┄┄┄┄┄┄┄┄┄├─────────────╊━━━━━━━━━━━━━━━━━━━╉───────────┤┄┄┄┄┄┄┄┄
           ╶┄start delay┄╴                   ╶┄end delay┄╴
                         ╶┄┄┄┄┄ duration┄┄┄┄┄╴

```


The [animation-range](#propdef-animation-range) properties are [reset-only sub-properties](https://www.w3.org/TR/css-cascade-5/#reset-only-sub-property) of the [animation](https://www.w3.org/TR/css-animations-1/#propdef-animation) [shorthand](https://www.w3.org/TR/css-cascade-5/#shorthand-property).

[](#issue-6cb6bdf3)Define application to time-driven animations.

#### Specifying an Animation’s Timeline Range: the [animation-range](#propdef-animation-range) shorthand[](#animation-range)


|Name:                 |animation-range                                              |
|----------------------|-------------------------------------------------------------|
|Value:                |[ <'animation-range-start'> <'animation-range-end'>? ]#      |
|Initial:              |see individual properties                                    |
|Applies to:           |see individual properties                                    |
|Inherited:            |see individual properties                                    |
|Percentages:          |see individual properties                                    |
|Computed value:       |see individual properties                                    |
|Animation type:       |see individual properties                                    |
|Canonical order:      |per grammar                                                  |


The [animation-range](#propdef-animation-range) property is a [shorthand](https://www.w3.org/TR/css-cascade-5/#shorthand-property) that sets [animation-range-start](#propdef-animation-range-start) and [animation-range-end](#propdef-animation-range-end) together in a single declaration, associating the animation with the specified [animation attachment range](#animation-attachment-range).

If [<'animation-range-end'>](#propdef-animation-range-end) is omitted and [<'animation-range-start'>](#propdef-animation-range-start) includes a [<timeline-range-name>](#typedef-timeline-range-name) component, then animation-range-end is set to that same <timeline-range-name> and 100%. Otherwise, any omitted [longhand](https://www.w3.org/TR/css-cascade-5/#longhand) is set to its [initial value](https://www.w3.org/TR/css-cascade-5/#initial-value).

[](#example-7788feda)The following sets of declarations show an [animation-range](#propdef-animation-range) [shorthand](https://www.w3.org/TR/css-cascade-5/#shorthand-property) declaration followed by its equivalent [animation-range-start](#propdef-animation-range-start) and [animation-range-end](#propdef-animation-range-end) declarations:

```
animation-range: entry 10% exit 90%;
animation-range-start: entry 10%;
animation-range-end: exit 90%;

animation-range: entry;
animation-range-start: entry 0%;
animation-range-end: entry 100%;

animation-range: entry exit;
animation-range-start: entry 0%;
animation-range-end: exit 100%;

animation-range: 10%;
animation-range-start: 10%;
animation-range-end: normal;

animation-range: 10% 90%;
animation-range-start: 10%;
animation-range-end: 90%;

animation-range: entry 10% exit;
animation-range-start: entry 10%;
animation-range-end: exit 100%;

animation-range: 10% exit 90%;
animation-range-start: 10%;
animation-range-end: exit 90%;

animation-range: entry 10% 90%;
animation-range-start: entry 10%;
animation-range-end: 90%;

```


[](#issue-e4fe9011)What’s the best way to handle defaulting of omitted values here? [\[Issue #8438\]](https://github.com/w3c/csswg-drafts/issues/8438)

#### Specifying an Animation’s Timeline Range Start: the [animation-range-start](#propdef-animation-range-start) property[](#animation-range-start)



* Name:      : Value:
    * animation-range-start     : [ normal | <length-percentage> | <timeline-range-name> <length-percentage>? ]#
* Name:      : Initial:
    * animation-range-start     : normal
* Name:      : Applies to:
    * animation-range-start     : all elements
* Name:      : Inherited:
    * animation-range-start     : no
* Name:      : Percentages:
    * animation-range-start     : relative to the specified named timeline range if one was specified, else to the entire timeline
* Name:      : Computed value:
    * animation-range-start     : list, each item either the keyword normal or a timeline range and progress percentage
* Name:      : Canonical order:
    * animation-range-start     : per grammar
* Name:      : Animation type:
    * animation-range-start     : not animatable


Specifies the start of the animations’s [attachment range](#animation-attachment-range), shifting the [start time](https://www.w3.org/TR/web-animations-1/#animation-start-time) of the animation (i.e. where keyframes mapped to 0% progress are attached when the iteration count is 1) accordingly.

Values have the following meanings:

normal

The start of the animation’s [attachment range](#animation-attachment-range) is the start of its associated [timeline](https://www.w3.org/TR/web-animations-1/#timeline); the start of the animation’s [active interval](https://www.w3.org/TR/web-animations-1/#active-interval) is determined as normal.

[<length-percentage>](https://www.w3.org/TR/css-values-4/#typedef-length-percentage)[](#valdef-animation-range-start-length-percentage)

The [animation attachment range](#animation-attachment-range) starts at the specified point on the [timeline](https://www.w3.org/TR/web-animations-1/#timeline) measuring from the start of the timeline.

[<timeline-range-name>](#typedef-timeline-range-name) [<length-percentage>](https://www.w3.org/TR/css-values-4/#typedef-length-percentage)?[](#valdef-animation-range-start-timeline-range-name-length-percentage)

The [animation attachment range](#animation-attachment-range) starts at the specified point on the [timeline](https://www.w3.org/TR/web-animations-1/#timeline) measuring from the start of the specified [named timeline range](#named-timeline-range). If the [<length-percentage>](https://www.w3.org/TR/css-values-4/#typedef-length-percentage) is omitted, it defaults to 0%.

#### Specifying an Animation’s Timeline Range End: the [animation-range-end](#propdef-animation-range-end) property[](#animation-range-end)



* Name:      : Value:
    * animation-range-end     : [ normal | <length-percentage> | <timeline-range-name> <length-percentage>? ]#
* Name:      : Initial:
    * animation-range-end     : normal
* Name:      : Applies to:
    * animation-range-end     : all elements
* Name:      : Inherited:
    * animation-range-end     : no
* Name:      : Percentages:
    * animation-range-end     : relative to the specified named timeline range if one was specified, else to the entire timeline
* Name:      : Computed value:
    * animation-range-end     : list, each item either the keyword normal or a timeline range and progress percentage
* Name:      : Canonical order:
    * animation-range-end     : per grammar
* Name:      : Animation type:
    * animation-range-end     : not animatable


Specifies the end of the animations’s [attachment range](#animation-attachment-range), potentially shifting the [end time](https://www.w3.org/TR/web-animations-1/#end-time) of the animation (i.e. where keyframes mapped to 100% progress are attached when the iteration count is 1) and/or truncating the animation’s [active interval](https://www.w3.org/TR/web-animations-1/#active-interval).

Values have the following meanings:

normal

The end of the animation’s [attachment range](#animation-attachment-range) is the end of its associated [timeline](https://www.w3.org/TR/web-animations-1/#timeline); the end of the animation’s [active interval](https://www.w3.org/TR/web-animations-1/#active-interval) is determined as normal.

[<length-percentage>](https://www.w3.org/TR/css-values-4/#typedef-length-percentage)[](#valdef-animation-range-end-length-percentage)

The [animation attachment range](#animation-attachment-range) ends at the specified point on the [timeline](https://www.w3.org/TR/web-animations-1/#timeline) measuring from the start of the timeline.

[<timeline-range-name>](#typedef-timeline-range-name) [<length-percentage>](https://www.w3.org/TR/css-values-4/#typedef-length-percentage)?[](#valdef-animation-range-end-timeline-range-name-length-percentage)

The [animation attachment range](#animation-attachment-range) ends at the specified point on the [timeline](https://www.w3.org/TR/web-animations-1/#timeline) measuring from the start of the specified [named timeline range](#named-timeline-range). If the [<length-percentage>](https://www.w3.org/TR/css-values-4/#typedef-length-percentage) is omitted, it defaults to 100%.

### Reporting Timeline Range Progress: the getCurrentTime() method[](#named-range-get-time)

Progress through named ranges is exposed on the `[AnimationTimeline](https://www.w3.org/TR/web-animations-1/#animationtimeline)` object by the `[getCurrentTime()](#dom-animationtimeline-getcurrenttime)` method:

```
dictionary AnimationTimeOptions {
  DOMString? range;
};

[Exposed=Window]
partial interface AnimationTimeline {
  CSSNumericValue? getCurrentTime(optional AnimationTimeOptions options = {});
};

```


`CSSNumericValue? getCurrentTime(optional AnimationCurrentTimeOptions = {})`

Returns a representation of the [current time](https://www.w3.org/TR/web-animations-1/#timeline-current-time) as follows:

If `[range](#dom-animationtimeoptions-range)` is not provided:

Returns the value of `[currentTime](https://www.w3.org/TR/web-animations-1/#dom-animationtimeline-currenttime)` on [this](https://webidl.spec.whatwg.org/#this), but representing millisecond values as a new `[CSSUnitValue](https://www.w3.org/TR/css-typed-om-1/#cssunitvalue)` in [ms](https://www.w3.org/TR/css-values-4/#ms) units rather than as a double.

If `[range](#dom-animationtimeoptions-range)` is provided and is a valid [named timeline range](#named-timeline-range) on [this](https://webidl.spec.whatwg.org/#this):

Let progress be the current progress through that range, expressed as a percentage value.

Create a [new unit value](https://drafts.css-houdini.org/css-typed-om-1/#create-a-cssunitvalue-from-a-pair) from (progress, "percent") and return it.

If the start and end points of the [named timeline range](#named-timeline-range) coincide, return negative infinity for time values earlier or equal to that point, and positive infinity for time values after it.

If `[range](#dom-animationtimeoptions-range)` is provided but is not a valid [named timeline range](#named-timeline-range) on [this](https://webidl.spec.whatwg.org/#this):

Returns null.

[](#issue-615b83f7)This method is related to `[currentTime](https://www.w3.org/TR/web-animations-1/#dom-animationtimeline-currenttime)` but not quite the same; should it have a different name? [\[Issue #8201\]](https://github.com/w3c/csswg-drafts/issues/8201)

[](#issue-8f9cc3f0)This method returns percentages relative to a ScrollTimeline’s range when a range name is provided. But for time-based timelines, if a range name is provided, should it return percentage progress through that range, or time progress through that range?

Appendix B: Timeline Name Scope[](#timeline-name-scope)
-------------------------------------------------------

[](#issue-be490712)This section should move to CSS-ANIMATIONS-2.

This appendix introduces the [timeline-scope](#propdef-timeline-scope) property, which allows declaring a timeline name’s scope on an ancestor of the timeline’s defining element.

### Declaring a Named Timeline’s Scope: the [timeline-scope](#propdef-timeline-scope) property[](#timeline-scope)


|Name:                 |timeline-scope                                     |
|----------------------|---------------------------------------------------|
|Value:                |none | <dashed-ident>#                             |
|Initial:              |none                                               |
|Applies to:           |all elements                                       |
|Inherited:            |no                                                 |
|Percentages:          |n/a                                                |
|Computed value:       |the keyword none or a list of CSS identifiers      |
|Canonical order:      |per grammar                                        |
|Animation type:       |not animatable                                     |


This property declares the scope of the specified timeline names to extend across this element’s subtree. This allows a named timeline (such as a [named scroll progress timeline](#named-scroll-progress-timelines) or [named view progress timeline](#named-view-progress-timelines)) to be referenced by elements outside the timeline-defining element’s subtree—for example, by siblings, cousins, or ancestors. It also blocks descendant timelines with the specified names from being referenced from outside this subtree, and ancestor timelines with the specified names from being referenced within this subtree.

[](#issue-73eb8d88)There’s some open discussion about these blocking effects. [\[Issue #8915\]](https://github.com/w3c/csswg-drafts/issues/8915)

Values have the following meanings:

none

No changes in timeline name scope.

[<dashed-ident>](https://www.w3.org/TR/css-values-4/#typedef-dashed-ident)[](#valdef-timeline-scope-dashed-ident)

Declares the name of a matching named timeline defined by a descendant—whose scope is not already explicitly declared by a descendant using [timeline-scope](#propdef-timeline-scope)—to be in scope for this element and its descendants.

If no such timeline exists, or if more than one such timeline exists, instead declares an [inactive timeline](https://www.w3.org/TR/web-animations-1/#inactive-timeline) with the specified name.

Note: This property cannot affect or invalidate any timeline name lookups within the subtree of a descendant element that declares the same name. See [Declaring a Named Timeline’s Scope: the timeline-scope property](#timeline-scope).

8\. Changes[](#changes)
-----------------------

Changes since the previous ([28 April 2023](https://www.w3.org/TR/2023/WD-scroll-animations-1-20230428/)) Working Draft include:

*   Removed scroll-timeline-attachment and view-timeline-attachment in favor of [timeline-scope](#propdef-timeline-scope). ([Issue 7759](https://github.com/w3c/csswg-drafts/issues/7759))

*   Switched named timelines to use [<dashed-ident>](https://www.w3.org/TR/css-values-4/#typedef-dashed-ident) instead of [<custom-ident>](https://www.w3.org/TR/css-values-4/#identifier-value) in order to avoid name clashes with standard CSS keywords. ([Issue 8746](https://github.com/w3c/csswg-drafts/issues/8746))


See also [Earlier Changes](https://www.w3.org/TR/2023/WD-scroll-animations-1-20230428/#changes).

Conformance requirements are expressed with a combination of descriptive assertions and RFC 2119 terminology. The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in the normative parts of this document are to be interpreted as described in RFC 2119. However, for readability, these words do not appear in all uppercase letters in this specification.

All of the text of this specification is normative except sections explicitly marked as non-normative, examples, and notes. [\[RFC2119\]](#biblio-rfc2119 "Key words for use in RFCs to Indicate Requirement Levels")

Examples in this specification are introduced with the words “for example” or are set apart from the normative text with `class="example"`, like this:

Informative notes begin with the word “Note” and are set apart from the normative text with `class="note"`, like this:

Note, this is an informative note.

Advisements are normative sections styled to evoke special attention and are set apart from other normative text with `<strong class="advisement">`, like this: **UAs MUST provide an accessible alternative.**

A style sheet is conformant to this specification if all of its statements that use syntax defined in this module are valid according to the generic CSS grammar and the individual grammars of each feature defined in this module.

A renderer is conformant to this specification if, in addition to interpreting the style sheet as defined by the appropriate specifications, it supports all the features defined by this specification by parsing them correctly and rendering the document accordingly. However, the inability of a UA to correctly render a document due to limitations of the device does not make the UA non-conformant. (For example, a UA is not required to render color on a monochrome monitor.)

An authoring tool is conformant to this specification if it writes style sheets that are syntactically correct according to the generic CSS grammar and the individual grammars of each feature in this module, and meet all other conformance requirements of style sheets as described in this module.

So that authors can exploit the forward-compatible parsing rules to assign fallback values, CSS renderers **must** treat as invalid (and [ignore as appropriate](https://www.w3.org/TR/CSS21/conform.html#ignore)) any at-rules, properties, property values, keywords, and other syntactic constructs for which they have no usable level of support. In particular, user agents **must not** selectively ignore unsupported component values and honor supported values in a single multi-value property declaration: if any value is considered invalid (as unsupported values must be), CSS requires that the entire declaration be ignored.

Once a specification reaches the Candidate Recommendation stage, non-experimental implementations are possible, and implementors should release an unprefixed implementation of any CR-level feature they can demonstrate to be correctly implemented according to spec.

To establish and maintain the interoperability of CSS across implementations, the CSS Working Group requests that non-experimental CSS renderers submit an implementation report (and, if necessary, the testcases used for that implementation report) to the W3C before releasing an unprefixed implementation of any CSS features. Testcases submitted to W3C are subject to review and correction by the CSS Working Group.
