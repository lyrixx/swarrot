# Swarrot

[![Build Status](https://travis-ci.org/swarrot/swarrot.png)](https://travis-ci.org/swarrot/swarrot)
[![Scrutinizer Quality Score](https://scrutinizer-ci.com/g/swarrot/swarrot/badges/quality-score.png?s=2c759b6224c762fc30a902d661b5512596060753)](https://scrutinizer-ci.com/g/swarrot/swarrot/)
[![SensioLabsInsight](https://insight.sensiolabs.com/projects/70007bd7-f9d8-460c-a35a-4e9fa1767ecb/mini.png)](https://insight.sensiolabs.com/projects/70007bd7-f9d8-460c-a35a-4e9fa1767ecb)

Swarrot is PHP library to consume messages from a broker.

## Installation

The recommended way to install Swarrot is through
[Composer](http://getcomposer.org/). Require the `swarrot/swarrot` package
into your `composer.json` file:

```json
{
    "require": {
        "swarrot/swarrot": "@stable"
    }
}
```

**Protip:** you should browse the
[`swarrot/swarrot`](https://packagist.org/packages/swarrot/swarrot)
page to choose a stable version to use, avoid the `@stable` meta constraint.

## Usage

### Basic usage

First, you need to create a message provider to retrieve message from you're
broker. For example, with A PeclPackageMessageProvider (retrieve message from
an AMQP broker with the [pecl amqp package](http://pecl.php.net/package/amqp):

    // Create connection
    $connection = new \AMQPConnection();
    $connection->connect();
    $channel = new \AMQPChannel($connection);
    // Get the queue to consume
    $queue = new \AMQPQueue($channel);
    $queue->setName('global');

    $messageProvider = new PeclPackageMessageProvider($queue);

Once it's done you need to create a Processor to process messages retrieved
from the broker. This processor can be a callback, a
`Swarrot\Processor\ProcessorInterface` or a
`Swarrot\Processor\Stack\StackedProcessor` (see [Using a
stack](#using-a-stack)). For the example, let's use a simple callback:

    $processor = function (Message $message, array $options) {
        echo sprintf("Consume message #%d\n", $message->getId());
    });

You now have a `Swarrot\Broker\MessageProviderInterface` to retrieve messages
and a Processor to process them. So, ask the `Swarrot\Consumer`to do it's job :

    $consumer = new Consumer($messageProvider, $processor);
    $consumer->consume();

### Decorate your processor

Using the [built in processors](#official-processors) or by [creating your
own](#create-your-own-processor), you can extend the bahavior of your
processor. Let's imagine you want to catch exception during execution to avoid
the consumer to stop in production environment, you can use the
[ExceptionCatcherProcessor](https://github.com/swarrot/exception-catcher-processor)
like this:

    use Swarrot\Processor\ExceptionCatcherProcessor;

    $myCallback = function (Message $message, array $options) {
        echo sprintf("Consume message #%d\n", $message->getId());
    });

    $processor = new ExceptionCatcherProcessor($myCallback);

Take a look at [this processor's
code](https://github.com/swarrot/exception-catcher-processor/blob/master/src/Swarrot/Processor/ExceptionCatcherProcessor.php#L23).
It just decorate your own processor with a try/catch block.

### Using a stack

Heavily inspired by [stackphp/builder](https://github.com/stackphp/builder) you
can use `Swarrot\Processor\Stack\Builder` to stack your processors.
Because it can be annoying to chain all you're processors, you can use the
Builder like this:

    $stack = (new \Swarrot\Processor\Stack\Builder())
        ->push('Swarrot\Processor\ExceptionCatcherProcessor')
        ->push('Swarrot\Processor\MaxMessagesProcessor', new Logger())
    ;

    $myCallback = function (Message $message, array $options) {
        echo sprintf("Consume message #%d\n", $message->getId());
    });

    $processor = $stack->resolve($myCallback);

## Processors

### Official processors

* [AckProcessor](https://github.com/swarrot/ack-processor)
* [ExceptionCatcherProcessor](https://github.com/swarrot/exception-catcher-processor)
* [InstantRetryProcessor](https://github.com/swarrot/instant-retry-processor)
* [MaxMessagesProcessor](https://github.com/swarrot/max-messages-processor) (thanks to [Remy Lemeunier](https://github.com/remyLemeunier))
* [SignalHandlerProcessor](https://github.com/swarrot/signal-handler-processor)

### Create your own processor

## Inspiration

* [stackphp/builder](https://github.com/stackphp/builder)

## License

Swarrot is released under the MIT License. See the bundled LICENSE file for details.
