examples privided by Andrey Kustov (G26)

parallelism options for node.js:
1. https://nodejs.org/api/cluster.html
2. pm2 start index.js -i max --name express_app


run:
```
pm2 start index.js -i max --name express_app
```

application code:

```
import crypto from "node:crypto";

import express from "express";

const port = 3001;
const app = express();

console.log(`worker pid=${process.pid}`);

app.get("/", (_, res) => res.send("Hello, world!\n"));

app.get('/delay/:msec', (req, res) => {
    const msec = parseInt(req.params.msec, 10);
    if (isNaN(msec) || msec < 0) {
        return res.status(400).send('Invalid msec parameter');
    }

    const initialTime = process.hrtime();
    const initialCpu = process.cpuUsage();

    let cycles = 0;
    let dummy = '';

    // Generate initial secure random number between 0-1
    const buffer = crypto.randomBytes(4);
    let number = buffer.readUInt32BE(0) / 0x100000000;

    do {
        // Perform CPU-intensive calculations
        number = number * Math.sqrt(Math.sin(number * Math.cos(number)));
        number = number / Math.sqrt(Math.cos(number + cycles + 1));

        dummy += number;

        // Update CPU time check
        const currentCpu = process.cpuUsage(initialCpu);
        const cpuTimeMicros = currentCpu.user + currentCpu.system;

        cycles++;

        if (cpuTimeMicros >= msec * 1000) {
            break;
        }
    } while (true);

    // Calculate final metrics
    const totalTime = process.hrtime(initialTime);
    const totalTimeMs = (totalTime[0] * 1000) + (totalTime[1] / 1e6);
    const cpuUsage = process.cpuUsage(initialCpu);
    const cpuTimeMs = (cpuUsage.user + cpuUsage.system) / 1000;
    const hashLength = dummy.length;

    const response =
        `Math ops cycles over 1K-string: ${cycles} \n` +
        `Real timer total: ${Math.round(totalTimeMs)} \n` +
        `Thread CPU : ${Math.round(cpuTimeMs)} \n` +
        `${hashLength}${dummy.charAt(hashLength - 1)}\n`;

    res.send(response);
});

app.listen(port, () => {
  console.log(`App listening on port ${port}`);
});

```
