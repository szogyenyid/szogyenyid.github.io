---
layout: post
title: "Writing sustainable PHP code: best practices and tools for energy-efficient programming"
date: 2024-03-17 11:35:00 +0100
categories:
    - sustainability
    - php
    - development
tags:
    - php
    - development
    - sustainability
    - webdev
    - energy-efficiency
    - green
description: "Discover how PHP developers can contribute to a greener digital ecosystem by adopting sustainable coding practices. This article explores energy-efficient programming techniques, introduces tools like Joulenne for energy consumption analysis, and highlights ecoCode's best practices. Dive into the world of sustainable PHP coding and pave the way for a more environmentally friendly future in software development."
#last_modified_at: 2024-03-17 11:35:00 +0100
author: Daniel Szogyenyi
readtime: 8
---

# Writing sustainable PHP code: best practices and tools for energy-efficient programming

## Introduction

In today's software development landscape, there's a rising awareness of the environmental impact of our code. Sustainable coding practices, emphasizing efficiency and resource optimization, have become essential. This article explores sustainable coding tailored for PHP developers, offering insights into best practices and tools to reduce the energy footprint of PHP applications.

As digital usage grows, so does energy consumption. Concerns over the environmental consequences urge developers to minimize energy usage while maintaining functionality and performance. Our focus here is on optimizing server-side PHP code for energy efficiency, empowering developers to create greener and more sustainable applications.

## Understanding sustainable coding in PHP

Sustainable PHP coding entails practices aimed at minimizing energy consumption and environmental impact while preserving functionality and performance. This involves optimizing code to reduce resource usage, including CPU cycles, memory, and network bandwidth, thereby promoting efficiency and sustainability in software development.

Inefficient PHP code not only affects functionality but also increases energy consumption and carbon emissions. Practices such as inefficient algorithms, suboptimal database queries, and redundant code execution exacerbate this impact, straining servers and infrastructure unnecessarily.

By adopting sustainable coding practices, PHP developers can significantly decrease energy consumption and resource usage. Optimization techniques like efficient algorithm design, caching mechanisms, and minimizing I/O operations not only enhance user experience but also lower server load and operational costs. Embracing these practices contributes to a greener digital ecosystem, aligning with broader efforts to mitigate the environmental impact of technology.

## Introducing an energy-efficiency analysis tool

In the quest for sustainable coding practices, accurately measuring energy consumption poses a significant challenge. Absolute energy consumption is influenced by various factors, including hardware infrastructure and environmental conditions, making it challenging to establish precise metrics. However, relative energy consumption offers a practical approach to comparing the efficiency of different code implementations under standardized conditions.

I've created a tool that facilitates the measurement of relative energy consumption: [Joulenne](https://github.com/szogyenyid/joulenne).  
Leveraging the Linux tool turbostat, Joulenne provides insights into the energy consumption of processes, offering valuable data for optimizing code efficiency. While turbostat being originally designed for general-purpose energy measurement, Joulenne allows it to be applied to any process, including PHP code execution.

Let's consider an example scenario where Joulenne is used to measure the energy consumption of a specific piece of PHP code. By executing the code on the same device and under consistent settings, Joulenne captures energy consumption metrics, providing developers with quantitative data to assess code efficiency.

### Example usage of [Joulenne](https://github.com/szogyenyid/joulenne)

Consider a typical bad practice in PHP: using `count($myArray)` in a for-loop.

You might have seen this code several times:
```php
for ($i=0; $i<count($a); $i++) {
    // Do something
}
```
It's straightforward, everyone understands is, but there's a problem; a for-loop checks the condition in the middle every cycle, and if it contains a function call, it gets called countless (no pun intended) times. But why would we count an array several times if it's content doesn't change? Let's do it better:  
```php
$c = count($a);
for ($i=0; $i<$c; $i++) {
    // Do something
}
```
But here's the question: __is it actually better__?

And this is the point where Joulenne comes in handy. Let's test [these examples](https://github.com/szogyenyid/joulenne/tree/main/examples/php/size-in-loop)! By putting these pieces of code in an infinite loop, and placing the files in a directory, we can use Joulenne to measure their energy usage:

```bash
sudo ./joulenne.sh --runner php --test-dir examples/php/size-in-loop --interval 1 --cycles 10
```

The `--interval` specifies the number of seconds a piece of code should run for, while `--cycles` tells the tool to run measurements this many times. So the above command runs every example code for 1 second, 10 times, and returns the average energy consumption of these 10 runs.

Here are my results on my computer:

| Test        | Energy consumption of CPU | Diff from sys | Compared to lowest |
| ----------- | ------------------------- | ------------- | ------------------ |
| sys         | 1.78 Joules               | 0             | -                  |
| before-loop | 2.57 Joules               | 0.79 Joules   | baseline           |
| in-loop     | 3.29 Joules               | 1.51 Joules   | +91.13%            |

Let's interpret this: if a for-loop with a `count()` in its check runs for a second, it will use about 91% more energy than a loop with a comparison to a predefined aim.

### Purpose and limitations of Joulenne

These energy-efficiency analysis tools work by monitoring hardware-level metrics, such as CPU utilization, frequency scaling, and power consumption, to evaluate code efficiency. By correlating code execution with energy consumption, developers can identify performance bottlenecks and inefficiencies, guiding them towards optimizing their applications for reduced resource usage and energy consumption.

Tools like this are not really able to monitor whole applications, as the majority of requests arriving to a server are handled differently. The main purpose of Joulenne is to identify best practices in lab-like environments, and apply them in real life after a lot of measurements and an exhaustive testing.

## Best practices for sustainable PHP coding

While there isn't a one-size-fits-all set of rules for writing energy-efficient PHP code, initiatives like [Green Code Initiative](https://github.com/green-code-initiative) aim to establish guidelines to promote sustainability in software development. Their tool, [ecoCode](https://github.com/green-code-initiative/ecoCode-php) has a collection of best practices specifically tailored for developers, designed to optimize code efficiency and reduce resource consumption.

Some of the key rules for PHP include:

- __Avoid getting size of collection in loops__: Minimize the retrieval of collection sizes within loops to reduce unnecessary overhead.
- __Avoid double quotes__: Favor the use of single quotes over double quotes for string literals to optimize CPU usage.
- __Avoid full SQL requests__: Optimize SQL queries by retrieving only the necessary data rather than fetching entire datasets.
- __Avoid SQL requests in loops__: Refrain from executing SQL queries within loops to prevent unnecessary database and network load.
- __Avoid multiple if-else statements__: Consolidate conditional statements to improve code readability and execution efficiency.

This is just a glimpse of the rules encapsulated by ecoCode-php. The complete list, along with code examples and integration resources, can be found in [their repository](https://github.com/green-code-initiative/ecoCode-php).

They not only provide guidelines for writing energy-efficient PHP code but also offers practical tools to facilitate adherence to these best practices. Its SonarQube plugin seamlessly integrates with development workflows, enabling developers to identify and address code inefficiencies directly within their IDE.

By adhering to ecoCode's principles and leveraging its resources, PHP developers can enhance the sustainability of their codebase, contributing to a more energy-efficient digital ecosystem.

## Conclusion and Call to Action

In conclusion, sustainable coding practices on the backend embody a commitment to environmental responsibility and resource optimization. By implementing energy-efficient techniques and utilizing tools like Joulenne and ecoCode-php, PHP developers can significantly reduce the carbon footprint of their applications.

Optimizing code for energy efficiency not only benefits the environment but also leads to improved performance, lower operational costs, and enhanced user experience. By embracing sustainable coding, developers showcase their dedication to environmental stewardship and inspire others in the software development community.

Let's recognize the impact of our code on the planet and prioritize sustainability in our coding philosophy. I urge PHP developers to adopt sustainable coding practices, integrate relevant tools, and contribute to a more energy-efficient future, one line of code at a time. Together, we can make a meaningful difference.