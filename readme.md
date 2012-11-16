
# CSS and Media Query Strategies

How do we create a CSS authoring strategy that:

* Allows non-media query capable browsers (such as IE) to get the "Desktop" version of the layout (making "mobile first" not an option)
* Avoids smaller form factors (eg. handheld) having to excessively override a desktop first strategy (making "desktop first" not an option)
* Lets CSS widgets and components be self contained with their rules grouped together for different targeted resolutions and dimensions
* Efficiently outputs media queries, collating together queries of the same type into single blocks rather than multiple repeating media query definitions
* Makes it easy to change the media query rules for retina triggering or other breakpoints
* Is transparent to developers and designers while authoring
* Maintains as much of the natural order of CSS as possible (eg. retain cascade order)
* Allows for the result to be efficiently concatenated and minified for optimal delivery over the wire

What follows is an exploration of using LESS to achieve as many of these goals as possible.

## Basic CSS Structure

The solution aims to map to the following broad (and deliberately verbose) over-arching structure, based on 320 and up:

	<!-- All browsers -->
	<link rel="stylesheet" href="shared.css">

	<!-- Mobile -->
	<link rel="stylesheet" media="only screen and (max-width: 767px)" href="mobile.css">

	<!-- Modern browsers (progressively larger resolutions) -->
	<link rel="stylesheet" media="only screen and (min-width: 320px)" href="320.css">
	<link rel="stylesheet" media="only screen and (min-width: 480px)" href="480.css">
	<link rel="stylesheet" media="only screen and (min-width: 768px)" href="768.css">
	<link rel="stylesheet" media="only screen and (min-width: 992px)" href="992.css">
	<link rel="stylesheet" media="only screen and (min-width: 1200px)" href="1200.css">
	<link rel="stylesheet" media="only screen and (min-width: 1320px)" href="1320.css">

	<!-- IE gets up to the 1024 styles -->
    <!--[if (lt IE 9)&(!IEMobile)]>
	<link rel="stylesheet" href="320.css">
	<link rel="stylesheet" href="480.css">
	<link rel="stylesheet" href="768.css">
	<link rel="stylesheet" href="992.css">
	<![endif]-->

Note: These files wouldn't be delivered to the user like this in the production result, it just illustrates the designers intent on how they'd like to mentally deal with the design strategy.

## Sample Component CSS

CSS rules for a single component can be grouped together, along with the design and layout adjustments for different media query breakpoints and resolutions.

	.selector {
		/* Universal rules for #selector go here, all browsers get these rules */
	}

	.320up() {
		.selector {
			/* Rules for any desired media query triggers go here, in this case it's a min-width: 320px rule */
		}
	}

	.768up() {
		.selector a {
			/* Additional rules. Not all defined trigger points need to be included, only where relevant */
		}
	}

	.all2x() {
		.selector {
			/* Rules for image overrides that apply at all dimensions for user agents that meet the @2x / hi-dpi rules */
		}
	}

	.768up2x() {
		.selector {
			/* Hi-dpi / @2x rules for the 768px breakpoint */
		}
	}

## Sample LESS Stucture

The output files are generated from the same source CSS, so there is no double maintenance or divergence of styles.

* `utilities.less` contains the media query trigger points, named media query variables and pre-defined output for legacy and modern styles
* `styles.less` is a demo file that represents your project styles. In practise, this would be a collection of project structured project files

### Legacy LESS

	@import "utilities";
	@import "styles";

	.legacyStyles;

### Modern LESS

	@import "utilities";
	@import "styles";

	.modernStyles;

## Sample CSS Output

Based on the example "sample component" code above, this is the resulting output.

### Legacy CSS

	.selector {
		/* Universal rules for #selector go here, all browsers get these rules */
	}
	.selector {
		/* Rules for any desired media query triggers go here, in this case it's a min-width: 320px rule */
	}
	.selector a {
		/* Additional rules. Not all defined trigger points need to be included, only where relevant */
	}

### Modern CSS

Note: Empty media query blocks have been removed in the example output to keep things focussed.

	.selector {
		/* Universal rules for #selector go here, all browsers get these rules */
	}

	@media only screen and (max-width: 767px) {
		.selector {
			/* Rules for any desired media query triggers go here, in this case it's a min-width: 320px rule */
		}
	}

	@media only screen and (min-width: 768px) {
		.selector a {
			/* Additional rules. Not all defined trigger points need to be included, only where relevant */
		}
	}

	@media only screen and (min-device-pixel-ratio: 1.3), only screen and (-webkit-min-device-pixel-ratio: 1.3), only screen and (-o-min-device-pixel-ratio: 13/10), only screen and (min-resolution: 120dpi) {
		.selector {
			/* Rules for image overrides that apply at all dimensions for user agents that meet the @2x / hi-dpi rules */
		}
	}

	@media only screen and (min-device-pixel-ratio: 1.3), only screen and (-webkit-min-device-pixel-ratio: 1.3), only screen and (-o-min-device-pixel-ratio: 13/10), only screen and (min-resolution: 120dpi), only screen and (min-width: 768px) {
		.selector {
			/* Hi-dpi / @2x rules for the 768px breakpoint */
		}
	}

## Caveats

The strategy outlined above meets all of the criteria listed at the start of the document, and provides a clean optimisation of grouping media queries together without repeating the same query over and over. There is however, one caveat that needs to be evaluated before adopting this strategy.

**While rules within media queries always retain their original order, the media query blocks (groups of rules) will always be output in the pre-defined media query group order**

This caveat is less about the technical implemention used, and more about the semantic debate of "grouped media queries" vs. "in-place / repeated media queries". If you're happily in the "grouped media queries" camp, then the caveat instead becomes a desirable feature.

To describe the situation in more detail:

* When CSS is authored, it honours the order of the rules when applying the cascade. If two rules are valid (and equally specific), the latter rule applies
* By grouping the media query rules into defined and ordered blocks (eg. `320 followed by 480, 768, 992, 1200, 1320`), rules in different media query blocks can be output in a different order than authored.

For example, when both of the below rules match, the final colour will be red:

	@media only screen and (min-width: 768px) {
		color: blue;
	}

	@media only screen and (min-width: 320px) {
		color: red;
	}

However, with media query grouping and pre-defined ordering, the rule set will be output in the order `320, 480, 768, 992, 1200, 1320`, regardless of the authored order. So the above rules would be renderers as follows, causing the final colour to be blue:

	@media only screen and (min-width: 320px) {
		color: red;
	}

	@media only screen and (min-width: 768px) {
		color: blue;
	}

**To avoid this sitation causing suprises, order your media query definitions in the same order as your predefined media query blocks. The resulting output will then semantically match the original cascade**

## Todo

* Do not write out any media query boilerplate for defined trigger points that are empty and unused
* Remove need for pre-defining media query variables by using a default / undefined check when outputting the rules