Controlling resourc allocation to a docker container : 

- find a script that can print cpu/cores and ram available

- run with docker till right configuration found

  ```
  docker run --cpuset-cpus=0 resources
  docker run -m=4m resources
  
  
  docker run resources --cpus=1
  
  sudo docker run -it --cpus="1.0" node:14
  
  docker run --cpuset-cpus=0 resources
  
  
  
  ```

  



### Controlling memory for a container

- source : https://docs.docker.com/config/containers/resource_constraints/

  

- | Option              | Description                                                  |
  | :------------------ | :----------------------------------------------------------- |
  | `-m` or `--memory=` | The maximum amount of memory the container can use. If you set this option, the minimum allowed value is `4m` (4 megabyte). |
  | `--memory-swap`*    | The amount of memory this container is allowed to swap to disk. See [`--memory-swap` details](https://docs.docker.com/config/containers/resource_constraints/#--memory-swap-details). |



### Configuring CPU

- 

  | Option                 | Description                                                  |
  | :--------------------- | :----------------------------------------------------------- |
  | `--cpus=<value>`       | Specify how much of the available CPU resources a container can use. For instance, if the host machine has two CPUs and you set `--cpus="1.5"`, the container is guaranteed at most one and a half of the CPUs. This is the equivalent of setting `--cpu-period="100000"` and `--cpu-quota="150000"`. |
  | `--cpu-period=<value>` | Specify the CPU CFS scheduler period, which is used alongside `--cpu-quota`. Defaults to 100000 microseconds (100 milliseconds). Most users do not change this from the default. For most use-cases, `--cpus` is a more convenient alternative. |
  | `--cpu-quota=<value>`  | Impose a CPU CFS quota on the container. The number of microseconds per `--cpu-period` that the container is limited to before throttled. As such acting as the effective ceiling. For most use-cases, `--cpus` is a more convenient alternative. |
  | `--cpuset-cpus`        | Limit the specific CPUs or cores a container can use. A comma-separated list or hyphen-separated range of CPUs a container can use, if you have more than one CPU. The first CPU is numbered 0. A valid value might be `0-3` (to use the first, second, third, and fourth CPU) or `1,3` (to use the second and fourth CPU). |
  | `--cpu-shares`         | Set this flag to a value greater or less than the default of 1024 to increase or reduce the container’s weight, and give it access to a greater or lesser proportion of the host machine’s CPU cycles. This is only enforced when CPU cycles are constrained. When plenty of CPU cycles are available, all containers use as much CPU as they need. In that way, this is a soft limit. `--cpu-shares` does not prevent containers from being scheduled in swarm mode. It prioritizes container CPU resources for the available CPU cycles. It does not guarantee or reserve any specific CPU access. |



A node script to get cpu cores; https://gist.github.com/jakoboo/82be8c031bc09cf2e75dac9253645f2a

```javascript
const os = require('os')
const childProcess = require('child_process')

function exec (command) {
  const output = childProcess.execSync(command, {encoding: 'utf8'})
  return output
}

let amount
const platform = os.platform()

if (platform === 'linux') {
  const output = exec('lscpu -p | egrep -v "^#" | sort -u -t, -k 2,4 | wc -l')
  amount = parseInt(output.trim(), 10)
} else if (platform === 'darwin') {
  const output = exec('sysctl -n hw.physicalcpu_max')
  amount = parseInt(output.trim(), 10)
} else if (platform === 'win32') {
  const output = exec('WMIC CPU Get NumberOfCores')
  amount = output.split(os.EOL)
    .map(function parse (line) { return parseInt(line) })
    .filter(function numbers (value) { return !isNaN(value) })
    .reduce(function add (sum, number) { return sum + number }, 0)
} else {
  const cores = os.cpus().filter(function (cpu, index) {
    const hasHyperthreading = cpu.model.includes('Intel')
    const isOdd = index % 2 === 1
    return !hasHyperthreading || isOdd
  })
  amount = cores.length
}

module.exports = amount
```