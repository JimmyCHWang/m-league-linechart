# m-league-linechart
Vue project to simulate M-league style stats line chart

---



## Overview

This project is a single page web application to simulate M-league[^1] style line chart results.

[^1]: M-league: A Mahjong TV series distributed by Abema.tv. 32 Japanese Mahjong professionals were divided into 8 teams and play a tournament league. 



## Usage

If you only plan to use without changing the source code, you could download the contents within the `dist` folder and put them on an arbitrary static content server.

Begin the game by changing the players' names using the yellow button on top-right.

When a hand is done, insert the four players' points to the form. In case the dealer has another hand, check the checkbox below. Press the blue button to add a record. If there is a mistake, use the red button can revert the result to the previous hand.

The chart below the form will renew once the blue / red button is pressed. 



Please note that this application uses no cache, so if you refresh the page, all records (such as the names, the records) will be reset to default. You may want to use this way to start a new game.



## Example

This example is generated from the 37th game of M-league Season 2021 (1st game on November 4th 2021). Special thanks to the owner of Twitter account "Mリーグ成績速報（非公式）"(@mleague_results) who collects the data.

[![sim-result.png](https://i.postimg.cc/85qLGQ6T/sim-result.png)](https://postimg.cc/wyQ15r2r)

The original line chart can be found here: https://abema.tv/video/episode/444-12_s85_p7 (4:04/13:57)



## Tips for Building From Source

The following tech stack is used within this project:

- Node.js
- Vue.js
  - Vue CLI (along with Babel/ESLint)
- Chart.js
- Bootstrap

---

### Installing the Environment

You will need *[Node.js](https://nodejs.org/)* (usually shorted as *Node*) for running environment along with its package manager *npm*. I used v14.17.5.

Once you installed *Node*, install *Vue CLI* as it is the official standard tool for *Vue* project development.

```shell
npm install -g @vue/cli
```

> You might need to run as Administrator / with `sudo`.
>
> Developers from Mainland China should try using `cnpm` instead of `npm` if the internet connection is slow. Check `cnpm` on arbitrary search engines to get help on installing `cnpm`.

Run `npm i` or `npm install` to install other dependencies of the project. 

You're all set.

---

### Running as Dev mode

```shell
npm run serve
```

If everything goes well, a localhost link will appear and click on that link to see the results.



### Build

```shell
npm run build
```

A folder `dist` will be generated once the build is finished and you are ready to deploy the project.



## Limitations

1. Currently it supports only Simplified Chinese. If you wish to have an English / Japanese version, try build one for yourself. I've already put the translations within comments beside the Chinese texts.
2. If you plan to build yourself, you might want to try turn on the animation of the chart (which is a feature of *Chart.js*), but it will crash if you tried to renew the chart before the ongoing animation is finished.
