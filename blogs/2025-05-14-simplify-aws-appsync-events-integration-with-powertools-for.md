---
title: "Simplify AWS AppSync Events integration with Powertools for AWS Lambda"
url: "https://aws.amazon.com/blogs/mobile/simplify-aws-appsync-events-integration-with-powertools-for-aws-lambda/"
date: "Wed, 14 May 2025 12:27:22 +0000"
author: "Ana Falcao"
feed_url: "https://aws.amazon.com/blogs/mobile/feed/"
---
<p>Real-time capabilities have become essential in modern applications, where users expect immediate updates and interactive experiences. Whether you’re building chat applications, live dashboards, gaming leaderboards, or IoT systems, <a href="https://docs.aws.amazon.com/appsync/latest/eventapi/event-api-welcome.html">AWS AppSync Events</a> enables these real-time features through WebSocket APIs, allowing you to build scalable and performant real-time applications, without worrying about scale or connection management.</p> 
<p><a href="https://powertools.aws.dev/">Powertools for AWS Lambda</a> is a developer toolkit that includes observability, batch processing, <a href="https://aws.amazon.com/systems-manager/">AWS Systems Manager</a> Parameter Store integration, idempotency, feature flags, <a href="https://aws.amazon.com/cloudwatch/">Amazon CloudWatch</a> Metrics, structured logging, and more. Powertools for AWS now supports AppSync Events through the new <code>AppSyncEventsResolver</code>, available in Python, TypeScript, and .NET. This new feature enhances the development experience with capabilities designed to help you focus on your business logic. The <code>AppSyncEventsResolver</code> provides a simple and consistent interface for processing events, with built-in support for common patterns such as filtering, transforming, and routing events.</p> 
<p>In this post, you will see examples in <a href="https://docs.powertools.aws.dev/lambda/typescript/latest/">TypeScript</a>, but you can use the same functionality in Python and .NET functions using <a href="https://docs.powertools.aws.dev/lambda/python/latest/">Powertools for AWS (Python)</a> and <a href="https://docs.powertools.aws.dev/lambda/dotnet/">Powertools for AWS (.NET)</a>.</p> 
<p><img alt="Real-time event handling architecture using AWS AppSync, Lambda, and Powertools" class="aligncenter wp-image-14101 size-full" height="412" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/13/pic1-1.png" width="936" /></p> 
<p style="text-align: center;">Figure 1 Real-time event handling architecture using AWS AppSync, Lambda, and Powertools</p> 
<p>In this post, you’ll learn how to:</p> 
<ul> 
 <li>Set up event handlers using the <code>AppSyncEventsResolver</code></li> 
 <li>Implement different event processing patterns for optimal performance</li> 
 <li>Use pattern-based routing to organize your event handlers</li> 
 <li>Leverage built-in features for common use cases</li> 
</ul> 
<h2>Getting Started</h2> 
<p>The <code>AppSyncEventsResolver</code> provides a simple, declarative way to handle AppSync Events in your <a href="https://aws.amazon.com/lambda/">AWS Lambda</a> functions. The event resolver allows you to listen for <code>PUBLISH</code> and <code>SUBSCRIBE</code> events. <code>PUBLISH</code> events occur when clients send messages to a channel, while <code>SUBSCRIBE</code> events happen when clients attempt to subscribe to a channel. You can register handlers for different namespaces and channels to manage your event-driven communications.</p> 
<p>Let’s explore how to get started and the core features that will enhance your development experience. Here’s a basic example of how to set up the <code>AppSyncEventsResolver</code>:</p> 
<pre><code class="lang-ts">import {
  AppSyncEventsResolver,
  UnauthorizedException,
} from '@aws-lambda-powertools/event-handler/appsync-events';

// Types for our message handling
type ChatMessage = {
    userId: string;
    content: string;
}

// Simple authorization check
const isAuthorized = (path: string, userId?: string): boolean =&gt; {
    // check against your authorization system
    if (path.startsWith('/chat/private') &amp;&amp; !userId) {
        return false;
    }
    return true;
};

// Message processing logic
const processMessage = async (payload: ChatMessage) =&gt; {
    // - Validate message content
    // - Store in database
    // - Enrich with additional data
    return {
        ...payload,
        timestamp: new Date().toISOString()
    };
};

const app = new AppSyncEventsResolver();

// Handle publish events for a specific channel
app.onPublish('/chat/general', async (payload: ChatMessage) =&gt; {
    // Process and return the message
    return processMessage(payload);
});

// Handle subscription events for all channels
app.onSubscribe('/*', async (info) =&gt; {
    const {
        channel: { path },
        request,
    } = info;

    // Perform access control checks
    if (!isAuthorized(path, userId)) {
        throw new UnauthorizedException(`not allowed to subscribe to ${path}`);
    }

    return true;
});

export const handler = async (event, context) =&gt;
  app.resolve(event, context);</code></pre> 
<p>The <code>AppSyncEventsResolver</code> class takes care of parsing the incoming event data and invoking the appropriate handler method based on the event type. Let’s break down what’s happening:</p> 
<h3>Pattern-based Routing</h3> 
<p>The <code>AppSyncEventsResolver</code> uses an intuitive pattern-based routing system that allows you to organize your event handlers based on channel paths. You can:</p> 
<ul> 
 <li>Handle specific channels (/chat/general)</li> 
 <li>Use wildcards for namespaces (/chat/*)</li> 
 <li>Create global catch-all handlers (/*)</li> 
</ul> 
<pre><code class="lang-ts">import { AppSyncEventsResolver } from '@aws-lambda-powertools/event-handler/appsync-events';

const app = new AppSyncEventsResolver();

// Specific channel handler
app.onPublish('/notifications/alerts', async (payload) =&gt; {
    // your logic here
});

// Handle all channels in the notifications namespace
app.onPublish('/notifications/*', async (payload) =&gt; {
    // your logic here
});

// Global catch-all for unhandled channels
app.onPublish('/*', async (payload) =&gt; {
    // your logic here
});

export const handler = async (event, context) =&gt;
  app.resolve(event, context);
</code></pre> 
<p>The most general catch-all handler is <code>/*</code>, which will match any namespace and channel, while <code>/default/*</code> will match any channel in the <code>default</code> namespace. When multiple handlers match the same event, the library will call the most specific handler and ignore the less specific ones. For example, if a handler is registered for <code>/default/channel1</code>&nbsp;and another one for <code>/default/*</code>, Powertools will call the first handler for events that match <code>/default/channel1</code>&nbsp;and ignore the second one. This provides you control over how events are handled and to avoid unnecessary processing. If an event that does not match any handler, by default Powertools will return the events as is and log a warning. This means that the events will be passed through without any modifications. This approach is helpful for gradually adopting the library, allowing you to handle specific events with custom logic while others are processed by the default behavior.</p> 
<h3>Subscription Handling</h3> 
<p>Powertools also provides a simple way to handle subscription events. It will automatically parse the incoming event and call the appropriate handler based on the event type. By default, AppSync allows the subscription unless your Lambda handler either throws an error or explicitly rejects the request. When a subscription is rejected, AppSync will return a 4xx response to the client and prevent the subscription from being established.</p> 
<pre><code class="lang-ts">import { AppSyncEventsResolver } from '@aws-lambda-powertools/event-handler/appsync-events';
import { Metrics, MetricUnit } from '@aws-lambda-powertools/metrics';
import type { Context } from 'aws-lambda';

const metrics = new Metrics({
  namespace: 'serverlessAirline',
  serviceName: 'chat',
  singleMetric: true,
});
const app = new AppSyncEventsResolver();

app.onSubscribe('/default/foo', (event) =&gt; {
  metrics.addDimension('channel', event.info.channel.path);
  metrics.addMetric('connections', MetricUnit.Count, 1);
});

export const handler = async (event: unknown, context: Context) =&gt;
  app.resolve(event, context);
</code></pre> 
<p>The library calls the appropriate handler and passes the event object as the first argument when a subscription event arrives. You can take any necessary actions based on the subscription event, such as running access control checks:</p> 
<pre><code class="lang-ts">app.onSubscribe('/private/*', async (info) =&gt; {
  const userGroups =
    info.identity?.groups &amp;&amp; Array.isArray(info.identity?.groups)
      ? info.identity?.groups
      : [];
  const channelGroup = 'premium-users';

  if (!userGroups.includes(channelGroup)) {
    throw new UnauthorizedException(
      `Subscription requires ${channelGroup} group membership`
    );
  }
})
</code></pre> 
<p>Subscription events follow the same matching rules and provide the same access to the full event and context. You can register catch-all handlers for any namespace or channel by using the wildcard <code>*</code>&nbsp;character, and also access the full event and context objects in your handlers.</p> 
<h3>Access full event and context</h3> 
<p>While the resolver simplifies event handling, you still have full access to the event and context objects when needed. This is helpful in scenarios where you need additional information, such as request headers or the remaining execution time from the Lambda context, to implement custom logic.</p> 
<p>The resolver passes the full event and context to each handler as the second and third arguments. This lets you access all relevant information without changing your existing code.</p> 
<pre><code class="lang-ts">import { AppSyncEventsResolver } from '@aws-lambda-powertools/event-handler/appsync-events';
import { Logger } from '@aws-lambda-powertools/logger';

const logger = new Logger({
  logLeveL: 'INFO',
  serviceName: 'serverlessAirline'
});
const app = new AppSyncEventsResolver({ logger});

app.onPublish('/orders/process', async (payload, event, context) =&gt; {
    // Access request headers
    const { headers } = event.request;
    
    // Access Lambda context
    const { getRemainingTimeInMillis } = context;

    logger.info('Processing event details', {
        headers,
        remainingTime: getRemainingTimeInMillis()
    });

    return payload;
});

export const handler = async (event, context) =&gt;
  app.resolve(event, context);
</code></pre> 
<h3>Error handling</h3> 
<p>The <code>AppSyncEventsResolver</code> offers built-in error handling that prevents Lambda function failures while ensuring errors are properly communicated to AppSync, which then propagates them to the clients. When an error occurs in your handler, instead of failing the entire Lambda invocation, the resolver captures the error and includes it in the response payload for the specific event that failed.</p> 
<p>This approach ensures the Lambda function continues execution while providing properly formatted error messages to AppSync. When processing multiple events, if one event fails, the others continue processing normally. This is particularly useful in parallel processing scenarios where you want to ensure that an error in one event doesn’t affect the processing of other events.</p> 
<pre><code class="lang-ts">import { AppSyncEventsResolver } from '@aws-lambda-powertools/event-handler/appsync-events';

const app = new AppSyncEventsResolver();

app.onPublish('/messages', async (payload) =&gt; {
    // If message contains "error", throw an exception
    if (payload.message === "error") {
        throw new Error("Invalid message");
    }
    return payload;
});

export const handler = async (event, context) =&gt;
  app.resolve(event, context);

// When processing this event:
// {
//     "id": "123",
//     "payload": {
//         "message": "error"
//     }
// }

// The resolver will return:
// {
//     "id": "123",
//     "error": "Error - Invalid message"
// }
</code></pre> 
<h2>Advanced Patterns and Best Practices</h2> 
<p>The <code>AppSyncEventsResolver</code> has additional advanced features that help you build robust and maintainable real-time applications. Let’s explore these capabilities and how to use them effectively.</p> 
<h3>On publish processing</h3> 
<p>By default, we call your route handler once per message. This allows you to focus on your business logic and avoid writing boilerplate code while Powertools handles message iteration and converts the event and response format. You only need to return the value you want to use as payload, or throw an error for that message. The library will then correlate the payload with the correct event id.</p> 
<pre><code class="lang-ts">import { AppSyncEventsResolver } from '@aws-lambda-powertools/event-handler/appsync-events';
import { Metrics, MetricUnit } from '@aws-lambda-powertools/metrics';

type SensorReading = {
    deviceId: string;
    temperature: number;
    humidity: number;
    timestamp: string;
}

const app = new AppSyncEventsResolver();
const metrics = new Metrics({ namespace: 'SensorReadings' });

app.onPublish('/sensors/readings', async (payload: SensorReading) =&gt; {
    // Process each sensor reading independently
    if (payload.temperature &gt; 100) {
        metrics.addDimension('alertType', 'highTemperature');
        metrics.addMetric('HighTemperature', MetricUnit.Count, 1);
        throw new Error('Temperature reading too high');
    }

    // Enrich the payload with processing timestamp
    return {
        ...payload,
        processed: true,
        processedAt: new Date().toISOString()
    };
});

export const handler = async (event, context) =&gt;
  app.resolve(event, context);
</code></pre> 
<p>This pattern simplifies development by letting you write only the logic for a single event, Powertools handles the rest automatically.</p> 
<h3>Aggregate Processing</h3> 
<p>The aggregate mode lets you to process multiple events as a single batch, rather than handling them individually. This is particularly useful when you want to optimize resource usage, such as sending multiple queries to a database in a single operation, or to analyze multiple events together before processing them. While both modes give you full control over event processing, aggregate mode provides access to the entire list of events at once.</p> 
<p>To achieve this, you can set the <code>aggregate</code> option to <code>true</code>. When using this mode, the resolver sends the entire list of events to your handler in a single call, letting you process them as a batch.</p> 
<pre><code class="lang-ts">import { AppSyncEventsResolver } from '@aws-lambda-powertools/event-handler/appsync-events';

const app = new AppSyncEventsResolver();

app.onPublish('/default/*', async (events) =&gt; {
  const results = [];
  for (const event of events) {
    try {
      results.push(await handleDefaultNamespaceCatchAll(event));
    } catch (error) {
      results.push({
        error: {
          errorType: 'Error',
          message: error.message,
        },
        id: event.id,
      });
    }
  }

  return results;
}, {
  aggregate: true,
});

export const handler = async (event, context) =&gt;
  app.resolve(event, context);
</code></pre> 
<p>Note that the <code>aggregate</code>&nbsp;option is only available for publish events, and that when using this option, you are responsible for handling the events and returning the appropriate response. Powertools will still take care of routing the events to the correct handler, but you have full control over how the events are processed.</p> 
<h3>Event Filtering</h3> 
<p>To filter out an event, throw an error in the channel handler. If a handler throws an error for a specific event, the library catches it and adds an error object to the response list at the same index. This signals that the corresponding message should be dropped. This allows you to either silently filter events or provide meaningful error feedback to your subscribers.</p> 
<pre><code class="lang-ts">import { AppSyncEventsResolver } from '@aws-lambda-powertools/event-handler/appsync-events';

app.onPublish('/moderation/*', async (payload) =&gt; {
    // Filter out inappropriate content
    if (await containsInappropriateContent(payload)) {
        throw new CustomError('Content violates guidelines');
    }

    // Process valid content
    return await processContent(payload);
});

export const handler = async (event, context) =&gt;
  app.resolve(event, context);</code></pre> 
<h2>Conclusion</h2> 
<p>The <code>AppSyncEventsResolver</code> in Powertools for AWS enhances your development experience with AppSync Events by providing a simple and consistent interface for processing real-time events. By reducing boilerplate code and offering built-in support for common patterns, you can focus on your business logic rather than infrastructure code.</p> 
<p>To learn more:</p> 
<ul> 
 <li>Explore the <a href="https://docs.powertools.aws.dev/lambda/typescript/latest/">Powertools for AWS documentation</a> for detailed information</li> 
 <li>Check out the <a href="https://docs.aws.amazon.com/appsync/latest/eventapi/event-api-welcome.html">AppSync Events documentation</a> to learn more about the service</li> 
 <li>Visit our <a href="https://github.com/aws-powertools">GitHub repository</a> to contribute or report issues</li> 
</ul> 
<p>We’re excited to see what you build with these new capabilities. Share your feedback and let us know how you’re using <code>AppSyncEventsResolver</code> in your applications!</p>
