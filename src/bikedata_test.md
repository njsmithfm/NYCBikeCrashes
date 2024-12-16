# Bike Crashes in Brooklyn
data from NYPD

```js echo
LineChart1=
Plot.plot({
  width: 1200, 
  height: 600, 
  margin: 40,
  color: { legend: false },
  marks: [
    Plot.line(Crashes, {
      x: "CRASH DATE",
      y: "NUMBER OF PERSONS INJURED",
      stroke: "ZIP CODE",
      z: "ZIP CODE",
      strokeWidth: 3,
      opacity: 0.15

    }),
      Plot.dotY(Crashes, { 
        x: "CRASH DATE",
        y: "NUMBER OF PERSONS INJURED",
        r: 2.5,
        fill: "ZIP CODE",
        tip: true
            }),
   
  ]
})
```

```table echo

```

```chart echo

```
