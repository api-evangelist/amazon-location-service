---
title: "Unlocking deeper insights with enhanced monitoring in AWS AppSync"
url: "https://aws.amazon.com/blogs/mobile/unlocking-deeper-insights-with-enhanced-monitoring-in-aws-appsync/"
date: "Thu, 09 Oct 2025 11:55:13 +0000"
author: "Sagar Suri"
feed_url: "https://aws.amazon.com/blogs/mobile/feed/"
---
<p>In today’s digital landscape, how well your APIs work is important for a good user experience. Monitoring your APIs isn’t just smart—it’s needed to keep your apps running well.</p> 
<p><a href="https://docs.aws.amazon.com/appsync/" rel="noopener" target="_blank">AWS AppSync</a>, a managed <a href="https://graphql.org/" rel="noopener" target="_blank">GraphQL</a> service, is a popular choice for developers building scalable, real-time applications. With the introduction of <a href="https://aws.amazon.com/about-aws/whats-new/2024/02/aws-appsync-amazon-cloudwatch-metrics-enhanced-monitoring/" rel="noopener" target="_blank">Enhanced Monitoring</a>, AppSync now provides deeper insights and more detailed metrics. In this blog post, we’ll show how this feature can help you monitor and optimise your APIs more effectively. We’ll also compare it to current monitoring tools, guide you through a sample API implementation, and break down the associated costs.</p> 
<h2>AWS AppSync overview</h2> 
<p>AWS AppSync simplifies the process of building and deploying secure, high-performance APIs. It’s a managed service for GraphQL and Pub/Sub APIs that abstracts away the complexities of infrastructure, allowing you to focus on what matters—building innovative applications.</p> 
<p>With AppSync, you can create a unified data graph that brings together data from multiple sources. Whether you’re synchronizing real-time data, enabling offline access, or managing conflict resolution, AppSync provides the tools to make it happen. Its built-in security features and serverless architecture mean you can deploy APIs that are both robust and secure, without worrying about the underlying infrastructure.</p> 
<h2>The importance of monitoring in API management</h2> 
<p>Monitoring is a critical aspect of API management. It ensures that APIs are performing optimally, meeting service level objectives, and maintaining security standards. By <a href="https://docs.aws.amazon.com/wellarchitected/latest/operational-excellence-pillar/ops_evolve_ops_metrics_review.html" rel="noopener" target="_blank">regularly keeping a close eye on key metrics</a>, you ensure that your APIs meet your service-level objectives (SLOs) and comply with security standards, ensuring a smooth and reliable experience. When you monitor metrics like latency, error rates, and request counts, you can detect anomalies early, troubleshoot root causes, and prevent small issues from escalating into user-facing problems. This approach of using telemetry from metrics to take proactive action aligns with best practices like <a href="https://docs.aws.amazon.com/wellarchitected/latest/operational-excellence-pillar/ops_operations_health_review_ops_metrics_prioritize_improvement.html" rel="noopener" target="_blank">OPS09-BP03</a> and is essential for the long-term reliability and scalability of your APIs.</p> 
<h2>Understanding existing monitoring in AWS AppSync</h2> 
<h3>Basic monitoring with AWS CloudWatch</h3> 
<p>AWS AppSync integrates with <a href="https://docs.aws.amazon.com/cloudwatch/" rel="noopener" target="_blank">Amazon CloudWatch</a>, providing essential metrics that offer a high-level overview of your API’s performance. These include request latency, counts and error rates and more.</p> 
<p>AppSync’s real-time subscriptions are a powerful feature, enabling push notifications and live updates. To monitor these, Amazon CloudWatch offers metrics such as the number of active real-time subscriptions, number of messages sent via subscription and more.</p> 
<p>Check the <a href="https://docs.aws.amazon.com/appsync/latest/devguide/monitoring.html" rel="noopener" target="_blank">full list of built-in metrics</a> to identify overall trends and spikes at a glance.</p> 
<h2>Introducing enhanced monitoring in AWS AppSync</h2> 
<p>As applications become more complex and user demands grow, there is a growing need for more granular insights. Developers increasingly require additional metrics to pinpoint the precise stages of request processing that might contribute to latency or errors.</p> 
<p>For instance, understanding the performance at the resolver level or tracking how individual data sources perform under load can be critical for fine-tuning and optimizing API operations. These detailed metrics empower developers to not only detect issues but also to understand the root causes, enabling more targeted and effective optimizations.</p> 
<p>On February 12, 2024, AWS announced <a href="https://aws.amazon.com/about-aws/whats-new/2024/02/aws-appsync-amazon-cloudwatch-metrics-enhanced-monitoring/" rel="noopener" target="_blank">Enhanced Monitoring for AWS AppSync</a>, Enhanced Monitoring in AWS AppSync gives developers access to a more detailed view of their API’s performance. This feature provides real-time insights and granular metrics, allowing you to drill down into specific operations, resolvers, and data sources.</p> 
<h3>Key features of enhanced monitoring</h3> 
<ul> 
 <li><strong>Resolver-Level Metrics:</strong> Track the performance of individual resolvers to pinpoint latency and errors. This allows for targeted optimizations, ensuring your API performs efficiently at every stage.</li> 
 <li><strong>Data Source Monitoring:</strong> Gain detailed insights into how your data sources—whether <a href="https://aws.amazon.com/dynamodb/" rel="noopener" target="_blank">Amazon DynamoDB</a>, <a href="https://aws.amazon.com/rds/" rel="noopener" target="_blank">Amazon RDS</a>, or <a href="https://aws.amazon.com/lambda/" rel="noopener" target="_blank">AWS Lambda</a>—are performing. Identifying performance bottlenecks at the source level becomes easier, allowing you to take corrective action quickly.</li> 
 <li><strong>Operational metrics:</strong> Get detailed metrics on GraphQL operation-level (Mutations &amp; Queries) metrics.</li> 
</ul> 
<p>In total, 10 new CloudWatch metrics were added under the <a href="https://docs.aws.amazon.com/appsync/latest/devguide/monitoring.html#enhanced-metrics" rel="noopener" target="_blank">“Enhanced metrics” section</a> of the official documentation.</p> 
<h2>Sample API implementation with enhanced monitoring</h2> 
<p>To demonstrate the power of Enhanced Monitoring, let’s walk through a sample AWS AppSync API implementation. This API will include three queries and two mutations, connected to four different data sources: Amazon DynamoDB, AWS Lambda, and Amazon OpenSearch Service.</p> 
<p>We will create an API that supports the following operations:</p> 
<p><strong>Queries:</strong></p> 
<ul> 
 <li><code class="inline">getUser</code>: Retrieve user details by ID.</li> 
 <li><code class="inline">listUsers</code>: List all users.</li> 
 <li><code class="inline">searchUsers</code>: Search users by a specific attribute.</li> 
</ul> 
<p><strong>Mutations:</strong></p> 
<ul> 
 <li><code class="inline">createUser</code>: Add a new user.</li> 
 <li><code class="inline">updateUser</code>: Update existing user details.</li> 
</ul> 
<p><strong>Data Sources:</strong></p> 
<ul> 
 <li><strong>DynamoDB</strong>: To store user data.</li> 
 <li><strong>AWS Lambda</strong>: For complex business logic processing (used for <code>createUser</code> and <code>updateUser</code> mutations).</li> 
 <li><strong>Amazon OpenSearch Service</strong>: To enable search functionality.</li> 
</ul> 
<h3>Step-by-step setup</h3> 
<h4>Prerequisites:</h4> 
<ul> 
 <li><strong>An active AWS account</strong>: If you don’t have one, you can <a href="https://portal.aws.amazon.com/billing/signup" rel="noopener" target="_blank">create an AWS account</a> to get started.</li> 
 <li><strong>Valid AWS credentials</strong>: You need an <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html" rel="noopener" target="_blank">IAM user</a> with permissions to manage AWS AppSync APIs (create, update, delete, view) and to view related CloudWatch metrics and alarms. This can be granted by attaching the AWS managed policies <code>AWSAppSyncAdministrator</code> and <code>CloudWatchReadOnlyAccess</code>, or equivalent custom permissions. You will use these credentials to sign in to <a href="https://console.aws.amazon.com/" rel="noopener" target="_blank">AWS Management Console</a>.</li> 
</ul> 
<p>Here’s how you can enable enhanced monitoring in AWS AppSync using different methods:</p> 
<h4>Method 1: Enabling enhanced monitoring via the AWS Management Console:</h4> 
<ol> 
 <li><strong>Navigate to the AppSync Console:</strong> 
  <ol> 
   <li>Sign in to the <a href="https://console.aws.amazon.com/" rel="noopener" target="_blank">AWS Management Console</a>.</li> 
   <li>Navigate to the <strong>AppSync</strong> service by searching for “AppSync” in the AWS services search bar.</li> 
  </ol> </li> 
 <li><strong>Select Your GraphQL API:</strong> 
  <ol> 
   <li>In the AppSync console, click on the API for which you want to enable enhanced monitoring.</li> 
  </ol> </li> 
 <li><strong>Go to the Settings:</strong> 
  <ol> 
   <li>Select “Settings” from the left-hand menu.</li> 
  </ol> </li> 
 <li><strong><strong><strong>Enable Enhanced Monitoring:</strong></strong></strong> 
  <div class="wp-caption alignnone" id="attachment_14278" style="width: 1847px;">
   <img alt="Screenshot of the AWS AppSync Console ‘Settings’ tab showing the toggles for Enhanced Monitoring. Three checkboxes are enabled: Per-Resolver Metrics, Per-Data Source Metrics, and Operation-Level Metrics." class="wp-image-14278 size-full" height="384" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/10/07/picture1-1.png" width="1837" />
   <p class="wp-caption-text" id="caption-attachment-14278">Figure 1: AWS AppSync Console ‘Settings’ tab showing the toggles for Enhanced Monitoring</p>
  </div> 
  <ol> 
   <li>In the API Configuration section, you’ll see an option to enable <strong>Enhanced Monitoring</strong>.</li> 
   <li>Here you can either enable metrics for all resolvers and data sources or can choose to manually select them by choosing “<code>PER_RESOLVER_METRICS</code>” and “<code>PER_DATA_SOURCE_METRICS</code>”, respectively, as stated in the documentation.</li> 
   <li>For per-resolver and per-data source metrics, metrics need to be enabled individually on each resolver and data source.</li> 
  </ol> </li> 
</ol> 
<h4>Method 2: Enabling enhanced monitoring via the AWS CLI</h4> 
<p><code>aws appsync update-graphql-api \<br /> </code><code>--api-id &lt;API_ID&gt; \<br /> </code><code>--name "&lt;API_NAME&gt;" \<br /> </code><code>--authentication-type API_KEY \<br /> </code><code>--enhanced-metrics-config resolverLevelMetricsBehavior=FULL_REQUEST_RESOLVER_METRICS,dataSourceLevelMetricsBehavior=FULL_REQUEST_DATA_SOURCE_METRICS,operationLevelMetricsConfig=ENABLE</code></p> 
<h4>Method 3: Enabling enhanced monitoring via Terraform</h4> 
<p>Following the <a href="https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/appsync_graphql_api#enhanced_metrics_config-block" rel="noopener" target="_blank">AWS Provider documentation</a>, adding the block <code>enhanced_metrics_config</code>to an existing resource will enable enhanced monitoring, eg:</p> 
<pre class="hl"><code>resource "aws_appsync_graphql_api" "api" {
  authentication_type = "API_KEY"
  name                = "&lt;API_NAME&gt;"
  enhanced_metrics_config {
    resolver_level_metrics_behavior     = "FULL_REQUEST_RESOLVER_METRICS"
    data_source_level_metrics_behavior  = "FULL_REQUEST_DATA_SOURCE_METRICS"
    operation_level_metrics_config      = "ENABLED"
  }
}
</code></pre> 
<h2>Metrics dashboard</h2> 
<p>With Enhanced Monitoring enabled, you can now observe detailed metrics in the Amazon CloudWatch dashboard. For example, if the SearchUsers query is experiencing high latency, you can quickly identify whether the issue lies with the Amazon OpenSearch Service data source or the Lambda resolver. The Amazon CloudWatch dashboard will display metrics such as resolver execution time and data source response time, allowing you to pinpoint the exact cause of any performance issues.</p> 
<h3>Additional API metrics in Amazon CloudWatch metrics</h3> 
<p>Useful dimensions and metrics include:</p> 
<ul> 
 <li>DataSource, GraphQLAPIId</li> 
 <li>API Metrics by Operation</li> 
 <li>GraphQLAPIId, Resolver</li> 
</ul> 
<div class="wp-caption alignnone" id="attachment_14279" style="width: 1891px;">
 <img alt="A screenshot of AWS AppSync API Metrics in Amazon CloudWatch" class="wp-image-14279 size-full" height="714" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/10/07/picture2.png" width="1881" />
 <p class="wp-caption-text" id="caption-attachment-14279">Figure 2: A screenshot of AWS AppSync API Metrics in Amazon CloudWatch</p>
</div> 
<h4>API Metrics per resolver</h4> 
<div class="figure"> 
 <div class="figcaption"> 
  <div class="wp-caption aligncenter" id="attachment_14281" style="width: 1891px;">
   <img alt="A screenshot of AWS AppSync API Metrics per-Resolver" class="wp-image-14281 size-full" height="719" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/10/07/image003.jpg" width="1881" />
   <p class="wp-caption-text" id="caption-attachment-14281">Figure 3: AWS AppSync API Metrics by Resolver</p>
  </div> 
 </div> 
</div> 
<div class="figure"> 
 <h4>API Metrics per DataSource</h4> 
 <div class="figcaption"> 
  <div class="wp-caption alignnone" id="attachment_14280" style="width: 1885px;">
   <img alt="A screenshot of AWS AppSync API Metrics per data source." class="wp-image-14280 size-full" height="717" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/10/07/image004.jpg" width="1875" />
   <p class="wp-caption-text" id="caption-attachment-14280">Figure 4: AWS AppSync API Metrics by Datasource</p>
  </div> 
 </div> 
 <h4>API Metrics per Operation</h4> 
 <div class="figcaption"> 
  <div class="wp-caption alignnone" id="attachment_14282" style="width: 1891px;">
   <img alt="A screenshot of AWS AppSync API Metrics per operation." class="wp-image-14282 size-full" height="718" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/10/07/image005.jpg" width="1881" />
   <p class="wp-caption-text" id="caption-attachment-14282">Figure 5: AWS AppSync API Metrics by Operation</p>
  </div> 
 </div> 
</div> 
<div class="figure"> 
 <div class="figcaption"></div> 
</div> 
<h3>API metrics custom dashboard with enhanced metrics</h3> 
<p>With AWS AppSync’s enhanced monitoring metrics, you can create a custom CloudWatch dashboard to get a real-time view of your API’s performance and display key KPIs like resolver latency, error rates, throttling, and data source performance. This helps quickly identify bottlenecks, track usage patterns, and optimise API efficiency.</p> 
<p>For example, you can set up widgets on your Amazon CloudWatch dashboard to monitor:</p> 
<ul> 
 <li><strong>Resolver Request Count:</strong> This will provide insights into usage patterns, guiding the team to optimise the most frequently used data sources and resolvers.</li> 
 <li><strong>Resolver Latency:</strong> Track the time taken by each resolver to process requests, helping you pinpoint areas that may require optimisation.</li> 
 <li><strong>Data Source Performance:</strong> Visualise the interaction between your API and its data sources, such as DynamoDB or Lambda, to ensure they are performing within acceptable thresholds.</li> 
 <li><strong>Error Rates:</strong> Display the number of errors encountered by each resolver or data source, enabling you to identify and address issues before they impact users.</li> 
</ul> 
<p>By aggregating these metrics into a single dashboard, you gain a holistic view of your API’s health, enabling proactive management and optimisation to deliver a seamless user experience.</p> 
<div class="wp-caption alignnone" id="attachment_14283" style="width: 1867px;">
 <img alt="A custom dashboard to show API Metrics by aggregating total request count, total Data source requests, average latencies per resolver and data source. Operation level metrics and overall API metrics. " class="wp-image-14283 size-full" height="744" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/10/07/image006.jpg" width="1857" />
 <p class="wp-caption-text" id="caption-attachment-14283">Figure 6: Custom Amazon CloudWatch dashboard to show API Metrics</p>
</div> 
<article class="post"> 
 <h2>Actionable insights from API monitoring</h2> 
 <p>Using the insights from an API metrics dashboard with enhanced metrics like those provided by AWS AppSync, you can take several actions to improve your API’s performance and reliability. Here are some specific actions you can take based on the metrics you monitor:</p> 
 <h3>High resolver request count</h3> 
 <ul> 
  <li><strong>Enable AppSync Caching:</strong> Use AppSync’s built-in caching to store results of frequently requested resolvers, reducing load on data sources. Set an appropriate TTL (Time to Live) for optimal balance between freshness and performance.</li> 
  <li><strong>Smaller Functional Units:</strong> Break down complex resolvers into smaller units. This improves visibility and makes it easier to identify and optimse specific tasks within your API.</li> 
 </ul> 
 <h3>High resolver latency</h3> 
 <ul> 
  <li><strong>Implement AppSync Caching:</strong> Cache responses for expensive queries to reduce latency for repeated requests.</li> 
  <li><strong>Optimise Resolvers:</strong> Refactor resolver logic to minimise processing time, such as reducing unnecessary computations or data transformations.</li> 
  <li><strong>Optimise Data Fetching:</strong> Ensure efficient database queries. Use indexes in DynamoDB and optimise SQL queries with proper indexing.</li> 
  <li><strong>Batch Operations:</strong> Use batch operations to minimise the number of data source calls. For example, use DynamoDB’s BatchGetItem for fetching multiple items in one request.</li> 
 </ul> 
 <h3>Poor data source performance</h3> 
 <ul> 
  <li><strong>Optimise AWS Lambda Functions:</strong> Increase memory allocation or refactor code for more efficient execution. Use Lambda’s metrics to identify performance issues.</li> 
  <li><strong>Use DynamoDB Accelerator (DAX):</strong> Integrate DAX to reduce read latency for high-traffic DynamoDB data sources by caching responses in-memory.</li> 
  <li><strong>RDS Optimisation:</strong> Utilise RDS read replicas for distributing read load and optimise queries with proper indexing.</li> 
 </ul> 
 <h3>Increased error rates</h3> 
 <ul> 
  <li><strong>Investigate and Resolve Issues:</strong> Use CloudWatch and AppSync logs to pinpoint and fix resolver or data source errors.</li> 
  <li><strong>Implement Retry Mechanisms:</strong> Use retry logic for transient errors or AWS SDK’s automatic retries to handle network interruptions.</li> 
  <li><strong>Improve Input Validation:</strong> Add validation to catch invalid or malformed requests early, preventing errors and providing informative messages to clients.</li> 
 </ul> 
 <h3>Frequent throttling occurrences</h3> 
 <ul> 
  <li><strong>Increase Throttling Limits:</strong> Adjust throttling limits in AppSync (using AWS WAF) or data sources like DynamoDB for legitimate high traffic.</li> 
  <li><strong>Optimise API Usage:</strong> Guide clients to reduce redundant requests and use efficient query structures.</li> 
  <li><strong>Implement Rate Limiting and Backoff:</strong> Apply rate limiting and exponential backoff on the client side to manage request rates and prevent hitting throttling limits.</li> 
 </ul> 
 <h2>Cost considerations</h2> 
 <p>Enhanced Monitoring emits additional metrics that incur charges in Amazon CloudWatch. Here’s a breakdown based on the sample API:</p> 
 <h3>Calculating costs — Example estimation using the sample API</h3> 
 <p><strong>Example calculation:</strong></p> 
 <p>Total no. of new metrics = <code class="inline">5 (3 queries + 2 mutations) * 5 (additional enhanced metrics)</code> + <code class="inline">4 (No. of Data sources) * 3 (additional enhanced metrics)</code> = <strong>37</strong></p> 
 <p>Monthly Amazon CloudWatch Metrics Charges @ <code class="inline">$0.30</code> per custom metric = <code class="inline">37 * $0.30 = $11.10</code></p> 
 <p>Note: Once you exceed 10,000 total metrics then volume pricing tiers will apply – To see the detailed pricing table, click the Metrics tab on the <a href="https://aws.amazon.com/cloudwatch/pricing/" rel="noopener" target="_blank">Amazon CloudWatch pricing page</a>.</p> 
 <h3>Reviewing costs</h3> 
 <p>Using <a href="https://docs.aws.amazon.com/cur/latest/userguide/what-is-data-exports.html" rel="noopener" target="_blank">AWS Data Exports</a> you can get an approximate cost breakdown of an existing AWS AppSync API by following these steps:</p> 
 <ol> 
  <li>Enable the enhanced monitoring temporarily on an AppSync API as seen above.</li> 
  <li>Execute queries or mutations against the AppSync API.</li> 
  <li><a href="https://docs.aws.amazon.com/cur/latest/userguide/dataexports-create-legacy.html" rel="noopener" target="_blank">Setup Cost Data Export</a></li> 
  <li><a href="https://docs.aws.amazon.com/cur/latest/userguide/dataexports-processing.html#dataexports-athena" rel="noopener" target="_blank">Use Amazon Athena to process Data Exports using the Crawlers</a>.</li> 
  <li>Run the Crawler (see <a href="https://docs.aws.amazon.com/athena/latest/ug/schema-crawlers.html" rel="noopener" target="_blank">Crawler docs</a>), and wait until it completes its run.</li> 
  <li>Replace with the Crawler’s target table.</li> 
  <li>Replace placeholders in the query below: 
   <ol> 
    <li><code>&lt;glue table&gt;</code>: Crawler’s target Glue table</li> 
    <li><code>&lt;account&gt;</code>: AWS account where AppSync is set up</li> 
    <li><code>&lt;region&gt;</code>: Region where AppSync is set up</li> 
   </ol> </li> 
  <li>Query Athena with the SQL query below by adjusting <code>line_item_usage_start_date</code>, <code>line_item_usage_end_date</code> to compare the following datasets: 
   <ol> 
    <li>During a timeframe when enhanced metrics were not yet used.</li> 
    <li>During a timeframe when enhanced metrics were already active.</li> 
   </ol> </li> 
 </ol> 
 <p>Sample Athena SQL (adjust placeholders):</p> 
 <pre class="hl"><code>SELECT
line_item_operation,
SUM(CAST(line_item_unblended_cost AS decimal(16,8))) AS TotalSpend
FROM ''
WHERE
product_product_name = 'AmazonCloudWatch'
AND line_item_usage_account_id = ''
AND product_region = ''
AND line_item_operation in ('MetricStorage:AWS/AppSync', 'GetMetricData', 'ListMetrics', 'GetMetricStatistics', 'PutMetricData')
AND line_item_line_item_type NOT IN ('Tax','Credit','Refund','EdpDiscount','Fee','RIFee')
AND line_item_usage_start_date &gt; DATE('2024-07-20')
AND line_item_usage_end_date &lt; DATE('2024-07-30')
GROUP BY line_item_operation
ORDER BY TotalSpend DESC;
</code></pre> 
 <p>Example output:</p> 
 <pre class="hl"><code># line_item_operation TotalSpend
1 GetMetricData 59.11682000
2 MetricStorage:AWS/AppSync 0.56626344
3 PutMetricData 0.42279000
4 ListMetrics 0.00248000
5 GetMetricStatistics 0.00028000
</code></pre> 
 <p>You can extract other details out of the data export by reviewing the <a href="https://docs.aws.amazon.com/cur/latest/userguide/Lineitem-columns.html" rel="noopener" target="_blank">Line item details</a> and <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_billing.html" rel="noopener" target="_blank">Usage types</a>.</p> 
 <h3>Metrics retention</h3> 
 <p><strong>CloudWatch retains metrics data as follows:</strong></p> 
 <p>Data points with a period of less than 60 seconds are available for 3 hours. These data points are high-resolution custom metrics.</p> 
 <ul> 
  <li>Data points with a period of 60 seconds (1 minute) are available for 15 days</li> 
  <li>Data points with a period of 300 seconds (5 minutes) are available for 63 days</li> 
  <li>Data points with a period of 3600 seconds (1 hour) are available for 455 days (15 months)</li> 
 </ul> 
 <p>Data points that are initially published with a shorter period are aggregated together for long-term storage. For example, if you collect data using a period of 1 minute, the data remains available for 15 days with 1-minute resolution. After 15 days this data is still available, but is aggregated and is retrievable only with a resolution of 5 minutes. After 63 days, the data is further aggregated and is available with a resolution of 1 hour.</p> 
 <h2>Conclusion</h2> 
 <p>AWS AppSync’s Enhanced Monitoring is a powerful tool that provides deeper insights into your API’s performance, delivering granular visibility to accelerate issue detection and remediation and reduce mean time to resolution. While there is an associated cost, the value it brings in terms of operational efficiency and improved user experience is substantial. By leveraging Enhanced Monitoring, you can ensure that your AWS AppSync APIs are optimised, reliable, and scalable.</p> 
 <p>Monitoring provides valuable data that empowers you to make informed decisions and drive your business forward. Ready to take your API performance to the next level? <strong>Activate Enhanced Monitoring in AWS AppSync</strong> for the detailed insights you need to keep your APIs running smoothly.</p> 
</article> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Sagar Suri" class="wp-image-11636 alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/10/06/1620923127007.jpeg" width="120" />
  </div> 
  <h3 class="lb-h4">Sagar Suri – Senior Delivery Consultant, AWS</h3> 
  <p style="text-align: left;">Sagar is a Senior Delivery Consultant at AWS, specialising in large-scale Cloud Architecture and governance. He leads strategic initiatives to help enterprise teams establish best practices, enabling the delivery of resilient, cost-optimised, and high-impact digital solutions across their application landscape.</p> 
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Bertram Varga" class="wp-image-11636 alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/10/09/vbertram-1.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Bertram Varga – Senior Delivery Consultant, AWS</h3> 
  <p style="text-align: left;">Bertram Varga is a Senior Cloud Application Architect at AWS Professional Services. He supports customers in building and modernizing applications in AWS in the Financial services, Automotive and Healthcare &amp; Life sciences domains.</p> 
 </div> 
</footer>
