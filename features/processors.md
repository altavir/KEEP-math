# Processor blocks for sequential values

* **Type**: Feature
* **Author**: Alexande Nozik
* **Status**: In progress
* **Discussion**: 

## Use-cases

* Signal processing
* Online signal analysis
* Univariate statistics
* Streaming data analysis
* [Maybe] Machine learning

## Feature description

An API allowing both stream-like and array-like processing of uniform univariate blocks of data (possibly generic). The processing must be organized in stages, where each stage could work stand-alone or attached to another stage. For example in time-series analysis one could want to first to perform moving window convolution, than apply filter and then apply zero suppression (get chunks where the signal value is larger than given value). The stages by default operate with suspending channels and push data model meaning that data is pushed into the first element and lazily propagates to the last. If at some moment processing capacity is reached and chain could no longer accept the element, pushing is suspended. 

The elements of the chain could be of three following types:

* Producer. Has an output `RecieveChannel` and no input. Could be easily wrapped around existing channel built by `produce` method.
* Processor. Has both input `SendChannel` and output `RecieveChannel`. The types of elements in both channels could be different. A Processor could require several elements in input to put something into output. Processor in general has internal changing state. Processor probably will implement both Producer and Consumer interface.
* Consumer. Has input `SendChannel` and internal state. The result of the chain is either represented by Consumer state that could be accessed from outside, or is calculated on-demand based on its state. It is possible for some consumers to have non-suspending accessible state. A Consumer could be wrapped around actor.

All chain elements have additional linking operation with another chain element. For example Producer could be linked with Processor or Consumer. Linked element automatically transmits its output to the element next in chain. An element with linked output could not be linked to another Consumer. 

The linking also allows to transfer data in more convenient format. For example if there is a Consumer that accepts Double, it could also accept DoubleArray. And Producer that generates DoubleArray could pass it directly, avoiding splitting it into separate numbers and boxing.

In general case the chain looks like this:
    `Producer -> Processor -> ... -> Processor -> Consumer`
The result is usually taken from consumer using its unique access method.
One could also use partial chains like:
    `Processor -> Consumer` or `Producer -> Processor`
putting or getting data using channels or specific methods of Processor. One could also manually redirect or change inputs and outputs of blocks possibly loosing benefits of optimized linking in the process.



