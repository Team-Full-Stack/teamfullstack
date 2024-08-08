---
layout: distill
title: Implementing Robust API Rate Limiting with ASP.NET Core
description: Learn how to implement API rate limiting in an ASP.NET Core application to manage request rates and ensure fair usage.
tags: ASP.NET Core, API Rate Limiting, Web Development
discus_comments: true
date: 2024-08-08
featured: false

authors:
  - name: Usman Jamil Bhatti
    url: "https://github.com/usman-jamil"
    affiliations:
      name: Allied Consultants
  - name:  Sheikh Luqman
    url: "https://github.com/luqmant51"
    affiliations:
      name: Allied Consultants
  - name: Zuhaib Sohail
    url: "https://github.com/ZuhaibSohail163"
    affiliations:
      name: Allied Consultants

bibliography: 2024-8-8-Api-Rate-Limit.bib

# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).

toc:
  - name: Introduction
  - name: Implementing Rate Limiting
    subsections:
      - name: Configuration
      - name: Rate Limiting Policies
      - name: Handling Rate Limit Exceed
  - name: Conclusion

_styles: >
  .fake-img {
    background: #bbb;
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 0px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 12px;
  }
  .fake-img p {
    font-family: monospace;
    color: white;
    text-align: left;
    margin: 12px 0;
    text-align: center;
    font-size: 16px;
  }
---

## Implementing Rate Limiting in ASP.NET Core

## Introduction

In modern web applications, managing API request rates is crucial for maintaining performance and ensuring fair usage among users. Implementing rate limiting helps to prevent abuse, mitigate denial-of-service attacks, and ensure that your service remains reliable for all users. In this blog post, we'll explore how to implement a robust API rate limiting system using ASP.NET Core.

## What is Rate Limiting?

Rate limiting is a technique used to control the number of requests a client can make to an API within a specified time frame. This is essential for preventing system overload, maintaining quality of service, and ensuring that no single client consumes too many resources, which could lead to degraded performance for others.

## How Does Rate Limiting Work?

Rate limiting can be implemented using various algorithms, each with its own approach to controlling the rate of requests. Common algorithms include:

- **Token Bucket**: Tokens are added to a bucket at a fixed rate, and each request consumes a token. If the bucket is empty, the request is denied or delayed.
- **Leaky Bucket**: Similar to Token Bucket, but with a focus on ensuring a steady outflow of requests, smoothing out bursts.
- **Fixed Window**: Counts the number of requests within a fixed time window (e.g., 1 minute) and enforces a limit.
- **Sliding Window**: Similar to Fixed Window, but the time window slides forward with each request, providing a more granular rate limit.

## Benefits of Rate Limiting

Implementing rate limiting provides several key benefits:

1. **Prevents Abuse**: Ensures that no single client can overwhelm the API with too many requests.
2. **Improves Performance**: Helps maintain consistent performance by controlling the load on the server.
3. **Enhances Security**: Mitigates denial-of-service (DoS) attacks by limiting the rate of incoming requests.
4. **Ensures Fair Usage**: Guarantees that resources are distributed fairly among users.

## Implementing Rate Limiting

To implement rate limiting in an ASP.NET Core application, follow these steps:

### 1. Add Rate Limiting Services

First, you need to add and configure the rate limiting services in your `Program.cs` file:

- You can download code from this repository [Api Rate Limiting](https://github.com/Luqmant51/RateLimit.Blog)

```csharp
builder.Services.AddRateLimiter(limiterOptions =>
{
    limiterOptions.AddPolicy("jwt", partitioner: httpContext =>
    {
        var accessToken = httpContext.Features.Get<IAuthenticateResultFeature>()?
                              .AuthenticateResult?.Properties?.GetTokenValue("access_token")
                          ?? string.Empty;

        if (!StringValues.IsNullOrEmpty(accessToken))
        {
            return RateLimitPartition.GetTokenBucketLimiter(accessToken, _ =>
                new TokenBucketRateLimiterOptions
                {
                    TokenLimit = 10,
                    QueueProcessingOrder = QueueProcessingOrder.OldestFirst,
                    QueueLimit = 0,
                    ReplenishmentPeriod = TimeSpan.FromSeconds(60),
                    TokensPerPeriod = 10,
                    AutoReplenishment = true,
                });
        }

        return RateLimitPartition.GetTokenBucketLimiter("Anon", _ =>
            new TokenBucketRateLimiterOptions
            {
                TokenLimit = 5,
                QueueProcessingOrder = QueueProcessingOrder.OldestFirst,
                QueueLimit = 0,
                ReplenishmentPeriod = TimeSpan.FromSeconds(60),
                TokensPerPeriod = 5,
                AutoReplenishment = true
            });
    });
    limiterOptions.OnRejected = (context, cancellationToken) =>
    {
        if (context.Lease.TryGetMetadata(MetadataName.RetryAfter, out TimeSpan retryAfter))
        {
            context.HttpContext.Response.Headers.RetryAfter = retryAfter.TotalSeconds.ToString(CultureInfo.InvariantCulture);
        }

        context.HttpContext.Response.StatusCode = StatusCodes.Status429TooManyRequests;
        context.HttpContext.Response.WriteAsync("Too many requests. Please try again later.", cancellationToken: cancellationToken);

        return new ValueTask();
    };
});
```

## What is a Partitioned Rate Limiter?
A partitioned rate limiter allows you to apply different rate limiting policies based on specific characteristics of incoming requests, such as user identity, IP address, or API key. This approach enables you to tailor rate limits to different groups of users or request types.

### `How Partitioning Works`
In the provided code example:

- Partitioning by Access Token:

   - If a request contains an access token, a specific rate limiting policy is applied based on that token (e.g., 10 requests per minute).
   - If no token is present (anonymous request), a different policy is applied (e.g., 5 requests per minute).

- Rate Limiting Policy:
  - Each partition has its own `TokenBucketLimiterOptions`, specifying the number of allowed requests, replenishment rate, and other settings.

- Handling Rate Limit Exceed:
  - When a rate limit is exceeded, the OnRejected callback sends a 429 Too Many Requests response, possibly including a Retry-After header.
  
- Benefits of Partitioned Rate Limiting
  - **Customization**: Custom rate limits for different user groups or request types.
  - **Fairness**: Ensures resource distribution is fair among users.
  - **Scalability**: Helps in scaling rate limiting strategies as your application grows.

This approach is particularly useful in scenarios where different users or client types require different levels of access, ensuring that your API remains responsive and reliable for all users.