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

[Scroll-progress-timelines] (#scroll-progress-timelines) также может быть определен на самом [scroll container](https://www.w3.org/TR/css-overflow-3/#scroll-container), а затем по имени обращаться к элементам в области видимости этого имени (см. [§ 4.2 Named Timeline Scoping and Lookup](#timeline-scoping)]).

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

Часто требуется, чтобы анимация начиналась и заканчивалась в тот отрезок времени [scroll progress timeline](#scroll-progress-timelines), когда определенный бокс (субъект прокрутки) находится в поле зрения [scrollport](https://www.w3.org/TR/css-overflow-3/#scrollport). Временные шкалы прокрутки - это сегменты временной шкалы прокрутки, которые привязаны к позициям прокрутки, в которых любая часть [основного бокса] элемента [https://www.w3.org/TR/css-display-3/#principal-box] пересекает ближайший предшествующий скроллпорт (точнее, соответствующий диапазон видимости [view progress visibility range](#view-progress-visibility-range) этого скроллпорта). Самая начальная позиция прокрутки представляет собой 0% прогресса, а самая конечная - 100% прогресса; смотрите [§ 3.2 Расчет прогресса для временной шкалы прогресса](#view-timeline-progress).

Примечание: Позиции прокрутки 0% и 100% не всегда достижимы, например, если окно расположено у начального края [scrollable overflow rectangle](https://www.w3.org/TR/css-overflow-3/#scrollable-overflow-rectangle), то прокрутка до прогресса < 32% может оказаться невозможной.

На [View progress timelines](#view-progress-timelines) можно ссылаться анонимно, используя [view()](#funcdef-view) [функциональную нотацию](https://www.w3.org/TR/css-values-4/#functional-notation) или по имени (см. [§ 4.2 Named Timeline Scoping and Lookup](#timeline-scoping)) после объявления их с помощью свойств [view-timeline](#propdef-view-timeline) на [view progress subject](#view-progress-subject). В Web Animations API они могут быть представлены анонимно объектом `[ViewTimeline](#viewtimeline)`.

### 3.1. Просмотр диапазонов временной шкалы прогресса

[View progress timelines](#view-progress-timelines) определяют следующие [named timeline ranges](#named-timeline-range):

cover

Представляет собой полный диапазон [view progress timeline](#view-progress-timelines):

* 0% прогресса представляет собой самую позднюю позицию, в которой [начало](https://dom.spec.whatwg.org/#concept-range-start) [край границы](https://www.w3.org/TR/css-box-4/#border-edge) [основного поля](https://www.w3.org/TR/css-display-3/#principal-box) элемента совпадает с [концом](https://dom.spec.whatwg.org/#concept-range-end) края его [диапазона видимости прогресса просмотра](#view-progress-visibility-range).

* 100% прогресс представляет собой самое раннее положение, при котором [конец](https://dom.spec.whatwg.org/#concept-range-end) [граничного края](https://www.w3.org/TR/css-box-4/#border-edge) [основного поля](https://www.w3.org/TR/css-display-3/#principal-box) элемента совпадает с [началом](https://dom.spec.whatwg.org/#concept-range-start) края его [диапазона видимости прогресса просмотра](#view-progress-visibility-range).


contain

Представляет собой диапазон, в котором [основной блок](https://www.w3.org/TR/css-display-3/#principal-box) либо полностью содержится, либо полностью перекрывает свой [диапазон видимости прогресса просмотра](#view-progress-visibility-range) в пределах [области прокрутки](https://www.w3.org/TR/css-overflow-3/#scrollport).

* 0 % прогресса - это самое раннее положение, в котором либо:

    * [начало](https://dom.spec.whatwg.org/#concept-range-start) [край границы](https://www.w3.org/TR/css-box-4/#border-edge) [основного поля](https://www.w3.org/TR/css-display-3/#principal-box) элемента совпадает с начальным краем его [диапазона видимости прогресса просмотра](#view-progress-visibility-range).

    * [конец](https://dom.spec.whatwg.org/#concept-range-end) [край границы](https://www.w3.org/TR/css-box-4/#border-edge) [основного поля](https://www.w3.org/TR/css-display-3/#principal-box) элемента совпадает с крайним краем его [диапазона видимости прогресса просмотра](#view-progress-visibility-range).

    * 100%-ный прогресс представляет собой последнюю позицию, в которой либо:

    * [начало](https://dom.spec.whatwg.org/#concept-range-start) [край границы](https://www.w3.org/TR/css-box-4/#border-edge) [основного бокса](https://www.w3.org/TR/css-display-3/#principal-box) элемента совпадает с начальным краем его [диапазона видимости прогресса просмотра](#view-progress-visibility-range).

    * [конец](https://dom.spec.whatwg.org/#concept-range-end) [граничный край](https://www.w3.org/TR/css-box-4/#border-edge) [основного поля](https://www.w3.org/TR/css-display-3/#principal-box) элемента совпадает с конечным краем его [диапазона видимости прогресса просмотра](#view-progress-visibility-range).


entry

Представляет собой диапазон, в котором [основной блок](https://www.w3.org/TR/css-display-3/#principal-box) входит в [диапазон видимости прогресса просмотра](#view-progress-visibility-range).

* 0% эквивалентно 0% диапазона [cover](#valdef-animation-timeline-range-cover).

* 100% эквивалентно 0% диапазона [contain](#valdef-animation-timeline-range-contain).

exit

Представляет собой диапазон, в котором [основной блок](https://www.w3.org/TR/css-display-3/#principal-box) выходит из [диапазона видимости прогресса просмотра](#view-progress-visibility-range).

* 0% эквивалентно 100% диапазона [contain](#valdef-animation-timeline-range-contain).

* 100% эквивалентно 100% диапазона [cover](#valdef-animation-timeline-range-cover).


entry-crossing

Представляет собой диапазон, в котором [основной блок](https://www.w3.org/TR/css-display-3/#principal-box) пересекает [конец](https://dom.spec.whatwg.org/#concept-range-end) [край границы](https://www.w3.org/TR/css-box-4/#border-edge)

* 0% прогресса представляет собой последнюю позицию, в которой [начало](https://dom.spec.whatwg.org/#concept-range-start) [край границы](https://www.w3.org/TR/css-box-4/#border-edge) [основного поля](https://www.w3.org/TR/css-display-3/#principal-box) элемента совпадает с [концом](https://dom.spec.whatwg.org/#concept-range-end) края его [диапазона видимости прогресса просмотра](#view-progress-visibility-range).

* 100% прогресс представляет собой самое раннее положение, при котором [конец](https://dom.spec.whatwg.org/#concept-range-end) [край границы](https://www.w3.org/TR/css-box-4/#border-edge) [основного поля](https://www.w3.org/TR/css-display-3/#principal-box) элемента совпадает с конечным краем его [диапазона видимости прогресса просмотра](#view-progress-visibility-range).


exit-crossing

Представляет собой диапазон, в котором [основной блок](https://www.w3.org/TR/css-display-3/#principal-box) пересекает [начало](https://dom.spec.whatwg.org/#concept-range-start) [край границы](https://www.w3.org/TR/css-box-4/#border-edge).

* 0% прогресса - это самое позднее положение, при котором [начало](https://dom.spec.whatwg.org/#concept-range-start) [край границы](https://www.w3.org/TR/css-box-4/#border-edge) [основного поля](https://www.w3.org/TR/css-display-3/#principal-box) элемента совпадает с начальным краем его [диапазона видимости прогресса просмотра](#view-progress-visibility-range).

* 100% прогресс представляет собой самое раннее положение, при котором [конец](https://dom.spec.whatwg.org/#concept-range-end) [край границы](https://www.w3.org/TR/css-box-4/#border-edge) [основного поля](https://www.w3.org/TR/css-display-3/#principal-box) элемента совпадает с [началом](https://dom.spec.whatwg.org/#concept-range-start) края его [диапазона видимости прогресса просмотра](#view-progress-visibility-range).


> Insert diagrams.

Во всех случаях для разрешения сторон [начало](https://dom.spec.whatwg.org/#concept-range-start) и [конец](https://dom.spec.whatwg.org/#concept-range-end) используется режим записи соответствующего [контейнера прокрутки](https://www.w3.org/TR/css-overflow-3/#scroll-container). [Трансформации](https://www.w3.org/TR/css-transforms/) игнорируются, но учитывается [относительное](https://www.w3.org/TR/CSS21/visuren.html#x34) и [абсолютное](https://www.w3.org/TR/css-position-3/#absolute-position) позиционирование.

> Примечание: Для [sticky-positioned boxes](https://www.w3.org/TR/css-position-3/#sticky-position) условия 0% и 100% прогресса иногда могут быть удовлетворены не одним, а целым рядом позиций прокрутки. Поэтому в каждом диапазоне указывается, какую позицию следует использовать - самую раннюю или самую позднюю.

[\[CSS-POSITION-3\]](#biblio-css-position-3 "CSS Positioned Layout Module Level 3") [\[CSS-TRANSFORMS-1\]](#biblio-css-transforms-1 "CSS Transforms Module Level 1")

### 3.2. Вычисление прогресса для временной шкалы прогресса

Прогресс ([текущее время](https://www.w3.org/TR/web-animations-1/#timeline-current-time)) в шкале времени [view progress timeline](#view-progress-timelines) вычисляется как: расстояние ÷ диапазон, где:

* расстояние - это текущее [смещение прокрутки](https://www.w3.org/TR/css-overflow-3/#scroll-offset) минус смещение прокрутки, соответствующее началу диапазона [cover](#valdef-animation-timeline-range-cover)

* range - это [scroll offset](https://www.w3.org/TR/css-overflow-3/#scroll-offset), соответствующее началу диапазона [cover](#valdef-animation-timeline-range-cover), минус scroll offset, соответствующее концу диапазона cover

Если позиции 0% и 100% совпадают (т.е. знаменатель в формуле [current time](https://www.w3.org/TR/web-animations-1/#timeline-current-time) равен нулю), то временная шкала является [неактивной](https://www.w3.org/TR/web-animations-1/#inactive-timeline).

В [paged media](https://www.w3.org/TR/mediaqueries-5/#paged-media) временные шкалы [view progress timelines](#view-progress-timelines), которые в противном случае ссылались бы на область просмотра документа, также являются [неактивными](https://www.w3.org/TR/web-animations-1/#inactive-timeline).

### 3.3. Анонимный просмотр графиков выполнения работ

#### 3.3.1. Нотация [view()](#funcdef-view)

Функциональная нотация view() может использоваться в качестве значения [<single-animation-timeline>](https://www.w3.org/TR/css-animations-2/#typedef-single-animation-timeline "Expands to: auto | none") в [animation-timeline](https://www.w3.org/TR/css-animations-2/#propdef-animation-timeline) и задает [view progress timeline](#view-progress-timelines) по отношению к ближайшему предку [scroll container](https://www.w3.org/TR/css-overflow-3/#scroll-container). Его синтаксис таков

```
<view()> = view( [ <axis> || <'view-timeline-inset'> ]? )
```

По умолчанию [view()](#funcdef-view) ссылается на [ось блока](https://www.w3.org/TR/css-writing-modes-4/#block-axis); как и для [scroll()](#funcdef-scroll), это можно изменить, указав явное значение [<axis>](#typedef-axis).

Необязательное значение [<'view-timeline-inset'>](#propdef-view-timeline-inset) обеспечивает настройку диапазона видимости [view progress visibility-range](#view-progress-visibility-range), как определено для view-timeline-inset.

Каждое использование [view()](#funcdef-view) соответствует собственному экземпляру `[ViewTimeline](#viewtimeline)` в Web Animations API, даже если несколько элементов используют view() для ссылки на один и тот же элемент с одинаковыми аргументами.

#### 3.3.2. Интерфейс `[ViewTimeline](#viewtimeline)`.

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

В качестве `[ViewTimeline](#viewtimeline)` выступает `[AnimationTimeline](https://www.w3.org/TR/web-animations-1/#animationtimeline)`, задающий временную шкалу [view progress timeline](#view-progress-timelines). Его можно передать конструктору `[Animation](https://www.w3.org/TR/web-animations-1/#animation)` или методу `[animate()](https://www.w3.org/TR/web-animations-1/#dom-animatable-animate)`, чтобы связать анимацию с временной шкалой прогресса просмотра.

`subject`, типа [Element](https://dom.spec.whatwg.org/#element), readonly

Элемент, видимость которого [principal box](https://www.w3.org/TR/css-display-3/#principal-box) в [scrollport](https://www.w3.org/TR/css-overflow-3/#scrollport) определяет ход временной шкалы.

`startOffset`, типа [CSSNumericValue](https://www.w3.org/TR/css-typed-om-1/#cssnumericvalue), readonly

Представляет начальную (0% прогресса) позицию прокрутки [view progress timeline](#view-progress-timelines) в виде смещения длины (в [px](https://www.w3.org/TR/css-values-4/#px)) от [scroll origin](https://www.w3.org/TR/css-overflow-3/#scroll-origin). Null, если временная шкала [неактивна](https://www.w3.org/TR/web-animations-1/#inactive-timeline).

`endOffset`, типа [CSSNumericValue](https://www.w3.org/TR/css-typed-om-1/#cssnumericvalue), readonly

Представляет конечную (100% прогресса) позицию прокрутки [view progress timeline](#view-progress-timelines) в виде смещения по длине (в [px](https://www.w3.org/TR/css-values-4/#px)) от [scroll origin](https://www.w3.org/TR/css-overflow-3/#scroll-origin). Null, если временная шкала [неактивна](https://www.w3.org/TR/web-animations-1/#inactive-timeline)..

> Примечание: Значения `[startOffset](#dom-viewtimeline-startoffset)` и `[endOffset](#dom-viewtimeline-endoffset)` относятся к [началу прокрутки](https://www.w3.org/TR/css-overflow-3/#scroll-origin), а не к [физическому](https://www.w3.org/TR/css-writing-modes-4/#physical) левому верхнему углу. Поэтому в зависимости от [режима записи](https://www.w3.org/TR/css-writing-modes-4/#writing-mode) [контейнера прокрутки](https://www.w3.org/TR/css-overflow-3/#scroll-container) они могут не совпадать со значениями `[scrollLeft](https://www.w3.org/TR/cssom-view-1/#dom-element-scrollleft)` или `[scrollTop](https://www.w3.org/TR/cssom-view-1/#dom-element-scrolltop)`, например, по [горизонтальной оси](https://www.w3.org/TR/css-writing-modes-4/#x-axis) в режиме записи справа налево ([rtl](https://www.w3.org/TR/css-writing-modes-4/#valdef-direction-rtl)).

Наследуемые атрибуты:

`source` (наследуется от `[ScrollTimeline](#scrolltimeline)`)

Ближайший предок `[subject](#dom-viewtimeline-subject)`, чей [основной блок](https://www.w3.org/TR/css-display-3/#principal-box) устанавливает [контейнер прокрутки](https://www.w3.org/TR/css-overflow-3/#scroll-container), позиция прокрутки которого управляет ходом временной шкалы.

`axis` (наследуется от `[ScrollTimeline](#scrolltimeline)`)

Определяет ось прокрутки, по которой движется временная шкала. См. раздел [<axis>](#typedef-axis), выше.

`currentTime` (наследуется от `[AnimationTimeline](https://www.w3.org/TR/web-animations-1/#animationtimeline)`)
Представляет текущий прогресс [view progress timeline](#view-progress-timelines) в процентах `[CSSUnitValue](https://www.w3.org/TR/css-typed-om-1/#cssunitvalue)`, отражающий прогресс прокрутки его [scroll container](https://www.w3.org/TR/css-overflow-3/#scroll-container) в данной позиции. Null, если временная шкала [неактивна](https://www.w3.org/TR/web-animations-1/#inactive-timeline).

`ViewTimeline(options)`

Создает новый объект `[ViewTimeline](#viewtimeline)` с помощью следующей процедуры:

1.  Пусть timeline - новый объект `[ViewTimeline](#viewtimeline)`.

2.  Установите для свойств `[subject](#dom-viewtimeline-subject)` и `[axis](#dom-scrolltimeline-axis)` timeline соответствующие значения из опций.

3.  Установите для свойства `[source](#dom-scrolltimeline-source)` временной шкалы значение `[subject](#dom-viewtimeline-subject)` ближайшего предка элемента [scroll container](https://www.w3.org/TR/css-overflow-3/#scroll-container).

4.  Если в качестве вставки предоставлено значение `[DOMString](https://webidl.spec.whatwg.org/#idl-DOMString)`, то оно разбирается как значение [<'view-timeline-inset'>](#propdef-view-timeline-inset); если предоставлена последовательность, то первое значение представляет собой начальную вставку, а второе - конечную. Если последовательность имеет только одно значение, то оно дублируется. Если она содержит нулевое значение или более двух значений, или если она содержит `[CSSKeywordValue](https://www.w3.org/TR/css-typed-om-1/#csskeywordvalue)`, чье `[value](https://www.w3.org/TR/css-typed-om-1/#dom-csskeywordvalue-value)` не является "auto", следует выбросить ошибку TypeError.

    Эти вставки определяют диапазон видимости `[ViewTimeline](#viewtimeline)` [view progress visibility range](#view-progress-visibility-range).

Если `[источник](#dom-scrolltimeline-source)` или `[субъект](#dom-viewtimeline-subject)` элемента `[ViewTimeline](#viewtimeline)` является элементом, [основной блок](https://www.w3.org/TR/css-display-3/#principal-box) которого не существует, или если его ближайший предок [контейнер прокрутки](https://www.w3.org/TR/css-overflow-3/#scroll-container) не имеет [scrollable overflow](https://www.w3.org/TR/css-overflow-3/#scrollable-overflow) (или если такого предка не существует, например, в печатных СМИ), то `[ViewTimeline](#viewtimeline)` является [неактивным](https://www.w3.org/TR/web-animations-1/#inactive-timeline).

Значения `[subject](#dom-viewtimeline-subject)`, `[source](#dom-scrolltimeline-source)` и `[currentTime](https://www.w3.org/TR/web-animations-1/#dom-animationtimeline-currenttime)` вычисляются при запросе или обновлении любого из них.

### 3.4. Именованные графики выполнения работ

[View progress timelines](#view-progress-timelines) также могут быть определены декларативно, а затем на них могут ссылаться по имени элементы в области видимости этого имени (см. [§ 4.2 Named Timeline Scoping and Lookup](#timeline-scoping)]).

Такие именованные временные шкалы выполнения вида объявляются в [согласованном списке значений](https://www.w3.org/TR/css-values-4/#coordinated-value-list), построенном из свойств view-timeline-\*, которые образуют группу свойств [согласованного списка](https://www.w3.org/TR/css-values-4/#coordinating-list-property) с [view-timeline-name](#propdef-view-timeline-name) в качестве базового свойства [согласованного списка](https://www.w3.org/TR/css-values-4/#coordinating-list-base-property). См. [CSS Values 4 § A Coordinating List-Valued Properties](https://www.w3.org/TR/css-values-4/#linked-properties).

#### 3.4.1. Именование временной шкалы прогресса представления: свойство [view-timeline-name](#propdef-view-timeline-name)

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

Определяет имена для [named view progress timelines](#named-view-progress-timelines), связанных с этим элементом.

#### 3.4.2. Ось временной шкалы представления: свойство [view-timeline-axis](#propdef-view-timeline-axis)

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

Определяет ось любого [named view progress timelines](#named-view-progress-timelines), полученного из [principal box](https://www.w3.org/TR/css-display-3/#principal-box) этого элемента.

Значения определяются для [view()](#funcdef-view).

#### 3.4.3. Вставка временной шкалы представления: свойство [view-timeline-inset](#propdef-view-timeline-inset)

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


Задает настройку [scrollport](https://www.w3.org/TR/css-overflow-3/#scrollport) на впуск (положительную) или выпуск (отрицательную) при определении того, находится ли бокс в поле зрения при установке границ соответствующей [view progress timeline](#view-progress-timelines). Первое значение представляет собой [начальный](https://dom.spec.whatwg.org/#concept-range-start) отступ по соответствующей оси; второе значение - [конечный](https://dom.spec.whatwg.org/#concept-range-end) отступ. Если второе значение опущено, то оно устанавливается равным первому. Полученный диапазон области прокрутки является диапазоном видимости прогресса просмотра.

**auto**

Указывает на использование значения параметра [scroll-padding](https://www.w3.org/TR/css-scroll-snap-1/#propdef-scroll-padding).

[<length-percentage>](https://www.w3.org/TR/css-values-4/#typedef-length-percentage)

Как и [scroll-padding](https://www.w3.org/TR/css-scroll-snap-1/#propdef-scroll-padding), задает смещение внутрь от соответствующего края области прокрутки.

#### 3.4.4. Сокращение временной шкалы просмотра: сокращение [view-timeline](#propdef-view-timeline)


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


Это свойство является [сокращением](https://www.w3.org/TR/css-cascade-5/#shorthand-property) для установки [view-timeline-name](#propdef-view-timeline-name) и [view-timeline-axis](#propdef-view-timeline-axis) в одном объявлении. Он не задает [view-timeline-inset](#propdef-view-timeline-inset).

> Должно ли оно также сбрасывать [view-timeline-inset](#propdef-view-timeline-inset)?

## 4.1 Присоединение анимации к временной шкале с прокруткой

Анимации могут быть привязаны к [scroll-driven timelines](#scroll-driven-timelines) с помощью свойства [scroll-timeline](#propdef-scroll-timeline) (в CSS) или параметров `[AnimationTimeline](https://www.w3.org/TR/web-animations-1/#animationtimeline)` (в Web Animations API). Диапазон временной шкалы, к которому привязан их [активный интервал](https://www.w3.org/TR/web-animations-1/#active-interval), также может быть дополнительно ограничен определенным диапазоном временной шкалы (см. раздел [Прикрепление анимаций к диапазонам временной шкалы](#named-range-animation-declaration)).

Задержки, основанные на времени ([animation-delay](https://www.w3.org/TR/css-animations-1/#propdef-animation-delay)), не применяются к [scroll-driven-animations](#scroll-driven-animations), которые основаны на расстоянии.

### 4.1. Расчеты на конечных временных интервалах

В отличие от временных шкал, [scroll-driven timelines](#scroll-driven-timelines) конечны, поэтому [scroll-driven-анимации](#scroll-driven-animations) всегда привязаны к конечному [attachment range](#animation-attachment-range)-который может быть дополнительно ограничен [animation-range](#propdef-animation-range) (см. [Appendix A: Timeline Ranges](#timeline-ranges)). Количество итераций ([animation-iteration-count](https://www.w3.org/TR/css-animations-1/#propdef-animation-iteration-count)) задается в пределах этого конечного диапазона. Если заданная длительность равна [auto](https://drafts.csswg.org/css-animations-2/#valdef-animation-duration-auto), то для нахождения [используемой](https://www.w3.org/TR/css-cascade-5/#used-value) длительности оставшийся диапазон делится на ее [количество итераций](https://www.w3.org/TR/web-animations-1/#iteration-count) (animation-iteration-count).
Note: If the animation has an infinite [iteration count](https://www.w3.org/TR/web-animations-1/#iteration-count), each [iteration duration](https://www.w3.org/TR/web-animations-1/#iteration-duration)—and the resulting [active duration](https://www.w3.org/TR/web-animations-1/#active-duration)—will be zero.

В анимации, включающей абсолютно позиционированные ключевые кадры (привязанные к определенной точке временной шкалы, например, с помощью [named timeline range keyframe selectors](#named-range-keyframes) в [@keyframes](https://www.w3.org/TR/css-animations-1/#at-ruledef-keyframes)), для определения положения ключевых кадров относительно 0% и 100% принимается [iteration count](https://www.w3.org/TR/web-animations-1/#iteration-count) равным 1; затем вся анимация масштабируется для соответствия [iteration duration](https://www.w3.org/TR/web-animations-1/#iteration-duration) и повторяется для каждой итерации.

Примечание: не совсем понятно, как можно использовать комбинацию абсолютно позиционированных ключевых кадров с числом итераций больше 1; это, по крайней мере, дает определенное поведение. (Альтернативным, но, возможно, более странным вариантом поведения было бы выведение таких абсолютно позиционированных ключевых кадров "из потока" при итерации остальных ключевых кадров). Редакторам было бы интересно узнать о реальных случаях использования нескольких итераций.

### 4.2. Именованная временная шкала и поиск

На именованный [scroll-progress-timelines](#scroll-progress-timelines) или [view progress timeline](#view-progress-timelines) можно ссылаться по:

* сам элемент, объявляющий имя

* потомки этого элемента


> Примечание: Свойство [timeline-scope](#propdef-timeline-scope) может быть использовано для объявления имени временной шкалы на предке определяющего ее элемента, эффективно расширяя ее область действия за пределы поддерева этого элемента.

Если в нескольких элементах объявлено одно и то же имя временной шкалы, то совпадающей временной шкалой считается та, которая объявлена в ближайшем по порядку дереве элементе. В случае конфликта имен в одном и том же элементе приоритет имеют имена, объявленные позже в свойстве именования ([scroll-timeline-name](#propdef-scroll-timeline-name), [view-timeline-name](#propdef-view-timeline-name)), а [scroll progress timelines](#scroll-progress-timelines) имеет приоритет над [view progress timelines](#view-progress-timelines).

> Используя timeline-scope, элемент может ссылаться на временные шкалы, привязанные к элементам, которые являются родными, двоюродными или даже потомками. Например, ниже создается анимация на элементе, который связан с [scroll progress timeline](#scroll-progress-timelines), определенным последующим братом или сестрой.
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


### 4.3. События анимации

[Анимации с прокруткой](#scroll-driven-animations) вызывают все те же события анимации, что и более типичные анимации, управляемые временем, как описано в [Web Animations § 4.4.18 Animation events](https://www.w3.org/TR/web-animations-1/#animation-events-section), [CSS Animations 1 § 4 Animation Events](https://www.w3.org/TR/css-animations-1/#events) и [CSS Animations 2 § 4.1 Event dispatch](https://www.w3.org/TR/css-animations-2/#event-dispatch).

Примечание: При прокрутке назад событие `animationstart` срабатывает в _конце_ [активного интервала](https://www.w3.org/TR/web-animations-1/#active-interval), а событие `animationend` - в начале активного интервала. Однако, поскольку событие `finish` связано с переходом в состояние [finished play state] (https://www.w3.org/TR/web-animations-1/#finished-play-state), оно срабатывает только при прокрутке вперед.

## 5. Детали расчета рамы
----------------------------------------

### 5.1. Модель обработки HTML: Цикл событий

Способность прокрутки управлять ходом анимации приводит к возникновению циклов компоновки, когда изменение смещения прокрутки приводит к обновлению эффекта анимации, что, в свою очередь, вызывает новое изменение смещения прокрутки.

Чтобы избежать таких циклов [layout cycles](#layout-cycles), анимации с [scroll progress timeline](#scroll-progress-timelines) обновляют свое текущее время один раз на шаге 7.10 цикла событий [HTML Processing Model](https://html.spec.whatwg.org/multipage/webappapis.html#processing-model-8), как шаг 1 [update animations and send events](https://www.w3.org/TR/web-animations-1/#update-animations-and-send-events).

На шаге 7.14.1 [HTML Processing Model](https://html.spec.whatwg.org/multipage/webappapis.html#processing-model-8) все созданные [scroll-progress timelines](#scroll-progress-timelines) или [view progress timelines](#view-progress-timelines) собираются в набор stale timelines. После шага 7.14, если [именованные диапазоны временных шкал](#named-timeline-range) каких-либо временных шкал изменились, эти шкалы добавляются в набор [stale timelines](#stale-timelines). Если есть устаревшие временные линии, то они теперь обновляют свое текущее время и связанные с ним диапазоны, набор устаревших временных линий очищается, и мы выполняем дополнительный шаг для пересчета стилей и обновления макета.

> Примечание: Мы проверяем изменения макета после отправки любого `[ResizeObserver](https://www.w3.org/TR/resize-observer-1/#resizeobserver)`, специально для того, чтобы учесть программные размеры элементов.

> Примечание: Поскольку мы собираем устаревшие временные шкалы только во время первого расчета стиля и макета, это может непосредственно вызвать только один дополнительный пересчет стиля. Другие API, требующие очередного обновления, должны быть проверены на том же этапе и обновлены одновременно.

> Примечание: Без этого дополнительного цикла пересчета стилей и компоновки [изначально несвежие](https://www.w3.org/TR/scroll-animations-1/#initially-stale) временные шкалы останутся несвежими (т.е. не будут иметь текущего времени) до конца кадра, в котором была создана временная шкала. Это означает, что анимация, связанная с такой временной шкалой, не будет создавать никаких эффектов для этого кадра, что может привести к нежелательной начальной "вспышке" в визуализации.

> Примечание: Данный раздел не влияет на принудительные вычисления стиля и компоновки, выполняемые с помощью `[getComputedStyle()](https://www.w3.org/TR/cssom-1/#dom-window-getcomputedstyle)` или аналогичных программ. Другими словами, [изначально неактуальные](https://www.w3.org/TR/scroll-animations-1/#initially-stale) временные шкалы видны как таковые через эти API.

Если окончательное обновление стиля и компоновки приведет к изменению времени или области видимости (см. [timeline-scope](#propdef-timeline-scope)) любых [scroll-progress-timelines](#scroll-progress-timelines) или [view progress timelines](#view-progress-timelines), то они не будут передискретизированы для отражения нового состояния до следующего обновления рендеринга.

Ничто в этом разделе не требует, чтобы прокрутка блокировалась в макете или скрипте. Если пользовательский агент обычно компонует кадры, в которых произошла прокрутка, но последствия прокрутки не были полностью отражены в макете или сценарии (например, еще не были запущены слушатели события `scroll`), то пользовательский агент может также не делать выборку анимации, управляемой прокруткой, для этого компонуемого кадра. В таких случаях отрисованное смещение прокрутки и состояние анимации, управляемой прокруткой, в скомпонованном кадре могут быть несовместимы.

## 6. Соображения конфиденциальности
-----------------------------------------------------

Неизвестно, какое влияние на конфиденциальность оказывают функции, представленные в данной спецификации.

## 7. Security Considerations
-------------------------------------------------------

О влиянии функций, представленных в данной спецификации, на безопасность ничего не известно.

Приложение A: Временные диапазоны
-----------------------------------------------

> Этот раздел следует перенести в разделы CSS-ANIMATIONS-2 и WEB-ANIMATIONS-2.

В этом приложении вводятся понятия [именованные диапазоны временной шкалы](#named-timeline-range) и [диапазоны вложений анимации](#animation-attachment-range) в [CSS Animations](https://www.w3.org/TR/css-animations/) и [Web Animations](https://www.w3.org/TR/web-animations/).

### Именованные диапазоны временной шкалы

Именованный диапазон временной шкалы - это именованный сегмент анимации [timeline](https://www.w3.org/TR/web-animations-1/#timeline). Начало сегмента представлено как 0% прогресса по диапазону; конец сегмента представлен как 100% прогресса по диапазону. С данной временной шкалой может быть связано несколько [именованных диапазонов временной шкалы](#named-timeline-range), причем несколько таких диапазонов могут пересекаться. Например, диапазон contain временной шкалы [view progress timeline](#view-progress-timelines) перекрывается с ее диапазоном cover. Именованные временные диапазоны представлены типом значения [<timeline-range-name>](#typedef-timeline-range-name), который указывает на [CSS-идентификатор](https://www.w3.org/TR/css-values-4/#css-css-identifier), представляющий один из предопределенных именованных временных диапазонов.

> Примечание: В данной спецификации [именованные диапазоны временной шкалы](#named-timeline-range) должны быть определены для существования спецификацией, такой как [\[SCROLL-ANIMATIONS-1\]](#biblio-scroll-animations-1 "Scroll-driven Animations"). На будущем уровне могут появиться API, позволяющие авторам объявлять свои собственные именованные диапазоны временной шкалы.

### Именованные селекторы ключевых кадров диапазона временной шкалы

Имена и проценты [Named timeline range](#named-timeline-range) могут быть использованы для прикрепления ключевых кадров к определенным точкам выполнения в пределах именованного временного диапазона. Правило CSS [@keyframes](https://www.w3.org/TR/css-animations-1/#at-ruledef-keyframes) расширяется таким образом:

```
<keyframe-selector> = from | to | <percentage [0,100]> | <timeline-range-name> <percentage>
```

где [<имя временного диапазона>](#typedef-timeline-range-name) - это [CSS-идентификатор](https://www.w3.org/TR/css-values-4/#css-css-identifier), который представляет выбранный предопределенный [именованный временной диапазон](#named-timeline-range), а [<процент>](https://www.w3.org/TR/css-values-4/#percentage-value) после него представляет собой процентное продвижение между началом и концом этого именованного временного диапазона.

Ключевые кадры привязываются к указанной точке временной шкалы. Если временная шкала не имеет соответствующего [named timeline range](#named-timeline-range), то все ключевые кадры, прикрепленные к точкам в этом именованном диапазоне временной шкалы, игнорируются. Возможно, что эти точки привязки находятся за пределами [активного интервала](https://www.w3.org/TR/web-animations-1/#active-interval) анимации; в этом случае автоматические ключевые кадры от (0%) и [до](https://drafts.csswg.org/css-shapes-2/#valdef-shape-to) (100%) генерируются только для тех свойств, которые не имеют ключевых кадров на или раньше 0% или на или позже 100% (соответственно).

### Прикрепление анимации к диапазонам временной шкалы

Набор ключевых кадров анимации может быть привязан к диапазону привязки анимации, ограничивая [активный интервал](https://www.w3.org/TR/web-animations-1/#active-interval) анимации этим диапазоном временной шкалы с помощью свойства [animation-range](#propdef-animation-range). Задержки (см. [animation-delay](https://www.w3.org/TR/css-animations-1/#propdef-animation-delay)) устанавливаются в этом ограниченном диапазоне, что еще больше сокращает время, доступное для [auto](https://drafts.csswg.org/css-animations-2/#valdef-animation-duration-auto) длительностей и [infinite](https://www.w3.org/TR/css-animations-1/#valdef-animation-iteration-count-infinite) итераций.

> Примечание: [animation-range](#propdef-animation-range) может как расширять диапазон [attachment range](#animation-attachment-range), так и сужать его.

Кадры, расположенные за пределами [диапазона вложения](#animation-attachment-range), используются для интерполяции по мере необходимости, но находятся за пределами [активного интервала](https://www.w3.org/TR/web-animations-1/#active-interval) и поэтому выпадают из самой анимации, фактически обрывая ее в конце диапазона вложения.

```
range start┐             ╺┉┉active interval┉┉╸           ┌range end
┄┄┄┄┄┄┄┄┄┄┄├─────────────╊━━━━━━━━━━━━━━━━━━━╉───────────┤┄┄┄┄┄┄┄┄
           ╶┄start delay┄╴                   ╶┄end delay┄╴
                         ╶┄┄┄┄┄ duration┄┄┄┄┄╴

```


Свойства [animation-range](#propdef-animation-range) являются [только сбрасываемыми подсвойствами](https://www.w3.org/TR/css-cascade-5/#reset-only-sub-property) [animation](https://www.w3.org/TR/css-animations-1/#propdef-animation) [shorthand](https://www.w3.org/TR/css-cascade-5/#shorthand-property).

Определите применение к анимации, управляемой временем.

#### Указание диапазона временной шкалы анимации: сокращение [animation-range](#propdef-animation-range)


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


Свойство [animation-range](#propdef-animation-range) - это [сокращение](https://www.w3.org/TR/css-cascade-5/#shorthand-property), которое задает [animation-range-start](#propdef-animation-range-start) и [animation-range-end](#propdef-animation-range-end) вместе в одном объявлении, связывая анимацию с указанным [диапазоном вложений анимации](#animation-attachment-range).

Если [<'animation-range-end'>](#propdef-animation-range-end) опущен и [<'animation-range-start'>](#propdef-animation-range-start) включает компонент [<timeline-range-name>](#typedef-timeline-range-name), то animation-range-end устанавливается на тот же <timeline-range-name> и 100%. В противном случае для любого опущенного [длинного диапазона](https://www.w3.org/TR/css-cascade-5/#longhand) устанавливается его [начальное значение](https://www.w3.org/TR/css-cascade-5/#initial-value).

Следующие наборы деклараций показывают декларацию [animation-range](#propdef-animation-range) [shorthand](https://www.w3.org/TR/css-cascade-5/#shorthand-property), за которой следуют эквивалентные ей декларации [animation-range-start](#propdef-animation-range-start) и [animation-range-end](#propdef-animation-range-end):

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


Как лучше всего обрабатывать умолчания о пропущенных значениях? [\[Issue #8438\]](https://github.com/w3c/csswg-drafts/issues/8438)

#### Указание начала временного диапазона анимации: свойство [animation-range-start](#propdef-animation-range-start)


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


Определяет начало диапазона [прикрепления](#animation-attachment-range) анимации, соответственно сдвигая [время начала](https://www.w3.org/TR/web-animations-1/#animation-start-time) анимации (т.е. место прикрепления ключевых кадров, сопоставленных с 0% прогресса, когда счетчик итераций равен 1).

Значения имеют следующие значения:

**normal**

Началом [диапазона вложений](#animation-attachment-range) анимации является начало связанной с ней [временной шкалы](https://www.w3.org/TR/web-animations-1/#timeline); начало [активного интервала](https://www.w3.org/TR/web-animations-1/#active-interval) анимации определяется обычным образом.

[<length-percentage>](https://www.w3.org/TR/css-values-4/#typedef-length-percentage)[](#valdef-animation-range-start-length-percentage)

Диапазон привязки [анимации](#animation-attachment-range) начинается в указанной точке на [временной шкале](https://www.w3.org/TR/web-animations-1/#timeline), измеряемой от начала временной шкалы.

[<timeline-range-name>](#typedef-timeline-range-name) [<length-percentage>](https://www.w3.org/TR/css-values-4/#typedef-length-percentage)?[](#valdef-animation-range-start-timeline-range-name-length-percentage)

Диапазон [прикрепления анимации](#animation-attachment-range) начинается в указанной точке на [временной шкале](https://www.w3.org/TR/web-animations-1/#timeline), отсчитываемой от начала указанного [именованного диапазона временной шкалы](#named-timeline-range). Если параметр [<длительность-процент>](https://www.w3.org/TR/css-values-4/#typedef-length-percentage) опущен, то по умолчанию он принимает значение 0%..

#### Указание конца временного диапазона анимации: свойство [animation-range-end](#propdef-animation-range-end)


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


Определяет конец диапазона привязки анимации [attachment range](#animation-attachment-range), потенциально смещая [end time](https://www.w3.org/TR/web-animations-1/#end-time) анимации (т.е. когда ключевые кадры, привязанные к 100% прогресса, привязываются при количестве итераций, равном 1) и/или усекая [active interval](https://www.w3.org/TR/web-animations-1/#active-interval) анимации.

Значения имеют следующие значения:

**normal**

Конец [диапазона вложений](#animation-attachment-range) анимации является концом связанной с ней [временной шкалы](https://www.w3.org/TR/web-animations-1/#timeline); конец [активного интервала](https://www.w3.org/TR/web-animations-1/#active-interval) анимации определяется обычным образом.

[<length-percentage>](https://www.w3.org/TR/css-values-4/#typedef-length-percentage)[](#valdef-animation-range-end-length-percentage)

Диапазон привязки [анимации](#animation-attachment-range) заканчивается в указанной точке на [временной шкале](https://www.w3.org/TR/web-animations-1/#timeline), отсчитываемой от начала временной шкалы.

[<timeline-range-name>](#typedef-timeline-range-name) [<length-percentage>](https://www.w3.org/TR/css-values-4/#typedef-length-percentage)?[](#valdef-animation-range-end-timeline-range-name-length-percentage)

Диапазон [прикрепления анимации](#animation-attachment-range) заканчивается в указанной точке [временной шкалы](https://www.w3.org/TR/web-animations-1/#timeline), отсчитываемой от начала указанного [именованного диапазона временной шкалы](#named-timeline-range). Если параметр [<длительность-процент>](https://www.w3.org/TR/css-values-4/#typedef-length-percentage) опущен, то по умолчанию он принимает значение 100%.

### Отчет о ходе выполнения диапазона временной шкалы: метод getCurrentTime()

Продвижение по именованным диапазонам осуществляется на объекте `[AnimationTimeline](https://www.w3.org/TR/web-animations-1/#animationtimeline)` методом `[getCurrentTime()](#dom-animationtimeline-getcurrenttime)`:

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

Возвращает представление [текущего времени](https://www.w3.org/TR/web-animations-1/#timeline-current-time) в следующем виде:

Если `[range](#dom-animationtimeoptions-range)` не указан:

Возвращает значение `[currentTime](https://www.w3.org/TR/web-animations-1/#dom-animationtimeline-currenttime)` на [this](https://webidl.spec.whatwg.org/#this), но представляет миллисекундные значения не в виде double, а в виде нового `[CSSUnitValue](https://www.w3.org/TR/css-typed-om-1/#cssunitvalue)` в единицах [ms](https://www.w3.org/TR/css-values-4/#ms).

Если указан `[диапазон](#dom-animationtimeoptions-range)` и он является допустимым [named timeline range](#named-timeline-range) на [this](https://webidl.spec.whatwg.org/#this):

Пусть progress - текущий прогресс по этому диапазону, выраженный в процентах.

Создать [новое значение единицы измерения](https://drafts.css-houdini.org/css-typed-om-1/#create-a-cssunitvalue-from-a-pair) из (progress, "percent") и вернуть его.

Если начальная и конечная точки [именованного диапазона временной шкалы](#named-timeline-range) совпадают, то для значений времени, более ранних или равных этой точке, возвращается отрицательная бесконечность, а для значений времени после нее - положительная бесконечность.

Если указано `[range](#dom-animationtimeoptions-range)`, но оно не является действительным [named timeline range](#named-timeline-range) на [this](https://webidl.spec.whatwg.org/#this):

Возвращает null.

> Этот метод связан с `[CurrentTime](https://www.w3.org/TR/web-animations-1/#dom-animationtimeline-currenttime)`, но не совсем то же самое; должно ли у него быть другое название? [\[Выпуск #8201\]](https://github.com/w3c/csswg-drafts/issues/8201)

Этот метод возвращает проценты относительно диапазона ScrollTimeline, если указано имя диапазона. Но для временных линий, основанных на времени, если указано имя диапазона, следует возвращать процентное продвижение по этому диапазону или временное продвижение по этому диапазону?

Приложение Б: Сроки Наименование Область применения
-------------------------------------------------------

> Этот раздел следует перенести в раздел CSS-ANIMATIONS-2.

В данном приложении введено свойство [timeline-scope](#propdef-timeline-scope), которое позволяет объявить область видимости имени временной шкалы на предке определяющего элемента временной шкалы.

### Объявление области действия именованной временной шкалы: свойство [timeline-scope](#propdef-timeline-scope)

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


Это свойство определяет область видимости указанных имен временных шкал, которая распространяется на все поддерево этого элемента. Это позволяет именованной временной шкале (например, [named scroll-progress timeline](#named-scroll-progress-timelines) или [named view progress timeline](#named-view-progress-timelines)) ссылаться на элементы вне поддерева определяющего временную шкалу элемента - например, на братьев, сестер, двоюродных братьев или предков. Также блокируются ссылки на временные шкалы потомков с указанными именами извне этого поддерева и ссылки на временные шкалы предков с указанными именами внутри этого поддерева.
[](#issue-73eb8d88)There’s some open discussion about these blocking effects. [\[Issue #8915\]](https://github.com/w3c/csswg-drafts/issues/8915)

Значения имеют следующие значения:

**none**

Изменений в области наименований сроков нет.

[<dashed-ident>](https://www.w3.org/TR/css-values-4/#typedef-dashed-ident)
Объявляет имя соответствующей именованной временной шкалы, определенной потомком, область видимости которой еще не объявлена потомком с помощью [timeline-scope](#propdef-timeline-scope), областью видимости для данного элемента и его потомков.

Если такой временной шкалы не существует или существует несколько таких шкал, то вместо нее объявляется [неактивная временная шкала](https://www.w3.org/TR/web-animations-1/#inactive-timeline) с указанным именем.

> Примечание: Это свойство не может повлиять на поиск имени временной шкалы в поддереве элемента-потомка, объявившего такое же имя, или сделать его недействительным. См. раздел [Объявление области действия именованной временной шкалы: свойство timeline-scope](#timeline-scope).
