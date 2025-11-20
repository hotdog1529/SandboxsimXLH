<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Optimized Sandbox Simulator</title>
<style>
    body {margin:0; background:#222; display:flex; flex-direction:column; align-items:center; font-family:sans-serif; user-select:none; color:white;}
    #ui {display:flex; gap:10px; padding:10px; flex-wrap:wrap; justify-content:center;}
    button, select, input {font-size:16px; padding:6px;}
    canvas {border:2px solid white; image-rendering:pixelated; touch-action:none;}
</style>
</head>
<body>

<div id="ui">
    <select id="material">
        <option value="1">Sand</option>
        <option value="2">Water</option>
        <option value="3">Lava</option>
        <option value="4">Smoke</option>
        <option value="5">Wall</option>
    </select>
    <label>Brush Size:
        <input id="brush" type="range" min="1" max="10" value="2">
    </label>
</div>

<canvas id="canvas"></canvas>

<script>
const CELL=3, W=200, H=200;
const canvas=document.getElementById("canvas");
canvas.width=W*CELL;
canvas.height=H*CELL;
const ctx=canvas.getContext("2d");

const EMPTY=0,SAND=1,WATER=2,LAVA=3,SMOKE=4,WALL=5,STONE=6;
let matColors={0:"#000",1:"#FFD27F",2:"#3FA9F5",3:"#FF5E3A",4:"#AAAAAA",5:"#777777",6:"#999966"};

let grid=new Array(W*H).fill(EMPTY);
let updated=new Array(W*H).fill(false);

let materialSelect=document.getElementById("material");
let brush=document.getElementById("brush");

function id(x,y){return y*W+x;}

// Place pixels based on brush size
function place(x,y){
    let b=parseInt(brush.value);
    for(let dy=-b;dy<=b;dy++){
        for(let dx=-b;dx<=b;dx++){
            let nx=x+dx, ny=y+dy;
            if(nx>=0&&nx<W&&ny>=0&&ny<H){
                grid[id(nx,ny)]=parseInt(materialSelect.value);
            }
        }
    }
}

// Input handlers
canvas.addEventListener("mousedown", e => handleInput(e.clientX,e.clientY));
canvas.addEventListener("mousemove", e => {if(e.buttons===1) handleInput(e.clientX,e.clientY);});
canvas.addEventListener("touchstart", e => {e.preventDefault();for(let t of e.touches) handleInput(t.clientX,t.clientY);});
canvas.addEventListener("touchmove", e => {e.preventDefault();for(let t of e.touches) handleInput(t.clientX,t.clientY);});

function handleInput(clientX, clientY){
    const r=canvas.getBoundingClientRect();
    let x=Math.floor((clientX-r.left)/CELL);
    let y=Math.floor((clientY-r.top)/CELL);
    place(x,y);
}

// Reset updated flags
function resetUpdated(){updated.fill(false);}

// Physics functions
function updateSand(x,y){
    let idx=id(x,y);
    if(y+1<H && (grid[id(x,y+1)]===EMPTY || grid[id(x,y+1)]===WATER || grid[id(x,y+1)]===SMOKE)){
        let below=id(x,y+1);
        [grid[idx],grid[below]]=[grid[below],grid[idx]];
    } else {
        if(x>0 && grid[id(x-1,y+1)]===EMPTY){let swap=id(x-1,y+1);[grid[idx],grid[swap]]=[grid[swap],grid[idx]];}
        else if(x<W-1 && grid[id(x+1,y+1)]===EMPTY){let swap=id(x+1,y+1);[grid[idx],grid[swap]]=[grid[swap],grid[idx]];}
    }
}

function updateWater(x,y){
    let idx=id(x,y);
    if(y+1<H && grid[id(x,y+1)]===EMPTY){let below=id(x,y+1);[grid[idx],grid[below]]=[grid[below],grid[idx]];}
    else{
        let dirs=[-1,1]; let d=dirs[Math.floor(Math.random()*2)];
        if(x+d>=0 && x+d<W && grid[id(x+d,y)]===EMPTY){let side=id(x+d,y);[grid[idx],grid[side]]=[grid[side],grid[idx]];}
    }
}

function updateSmoke(x,y){
    let idx=id(x,y);
    if(y>0 && grid[id(x,y-1)]===EMPTY){let up=id(x,y-1);[grid[idx],grid[up]]=[grid[up],grid[idx]];}
}

function updateLava(x,y){
    let idx=id(x,y);
    if(updated[idx]) return;
    let below=id(x,y+1);
    if(y+1<H){
        if(grid[below]===EMPTY){
            grid[below]=LAVA; grid[idx]=EMPTY; updated[below]=true; return;
        }
        if(grid[below]===WATER){grid[below]=STONE; grid[idx]=EMPTY; updated[below]=true; return;}
    }
    // Spread sideways
    let dir=Math.random()<0.5?-1:1;
    let side1=id(x+dir,y), side2=id(x-dir,y);
    if(x+dir>=0 && x+dir<W && grid[side1]===EMPTY){grid[side1]=LAVA; grid[idx]=EMPTY; updated[side1]=true; return;}
    if(x-dir>=0 && x-dir<W && grid[side2]===EMPTY){grid[side2]=LAVA; grid[idx]=EMPTY; updated[side2]=true; return;}
    updated[idx]=true;
}

// Main simulation step
function step(){
    resetUpdated();
    for(let y=H-2;y>=0;y--){
        for(let x=0;x<W;x++){
            let idx=id(x,y), cell=grid[idx];
            if(cell===SAND) updateSand(x,y);
            if(cell===WATER) updateWater(x,y);
            if(cell===LAVA) updateLava(x,y);
            if(cell===SMOKE) updateSmoke(x,y);
        }
    }
}

// Draw function
function draw(){
    for(let y=0;y<H;y++){
        for(let x=0;x<W;x++){
            ctx.fillStyle=matColors[grid[id(x,y)]]||"#000";
            ctx.fillRect(x*CELL,y*CELL,CELL,CELL);
        }
    }
}

// Main loop
function loop(){
    step();
    draw();
    requestAnimationFrame(loop);
}
loop();

</script>
</body>
</html>
