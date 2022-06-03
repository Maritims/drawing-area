<script lang="ts">
    type CanvasGridSettings = {
        width: number,
        height: number,
        scale: number,
        radius: number,
        clearButtonText: string,
        undoButtonText: string,
        redoButtonText: string,
        saveButtonText: string
    };

    export let settings: CanvasGridSettings = {
        width: 10,
        height: 2,
        scale: 150,
        radius: 2,
        clearButtonText: 'Clear',
        undoButtonText: 'Undo',
        redoButtonText: 'Redo',
        saveButtonText: 'Save'
    }
    
    let canvas: HTMLCanvasElement;
    let isDrawing = false;
    const start = 0;
    const end = Math.PI * 2;

    let undoList: string[] = [];
    let redoList: string[] = [];

     $: context = canvas ? canvas.getContext('2d') : undefined;

    const draw = (e: MouseEvent): void => {
        if(!isDrawing) return;

        context.lineTo(e.offsetX, e.offsetY);
        context.stroke();
        context.beginPath();
        context.arc(e.offsetX, e.offsetY, settings.radius, start, end);
        context.fill();
        context.beginPath();
        context.moveTo(e.offsetX, e.offsetY);
    }

    const startDrawing = (e: MouseEvent): void => {
        isDrawing = true;
        undoList.push(canvas.toDataURL());
        draw(e);
    }

    const stopDrawing = (e: MouseEvent): void => {
        isDrawing = false;
        context.beginPath();
    }

    const restoreState = (popFromList: string[], pushToList: string[]): void => {
        if(!popFromList.length) return;

        pushToList.push(canvas.toDataURL());
        const img = new Image();
        img.src = popFromList.pop();
        img.onload = () => {
            context.clearRect(0, 0, canvas.width, canvas.height);
            context.drawImage(img, 0, 0, canvas.width, canvas.height, 0, 0, canvas.width, canvas.height);
        }
    }

    const undo = (): void => restoreState(undoList, redoList);

    const redo = (): void => restoreState(redoList, undoList);

    const clear = (): void => {
        context.fillStyle = 'white';
        context.fillRect(0, 0, canvas.width, canvas.height);
        context.fillStyle = 'black';
        undoList = [];
        redoList = [];
    }

    const save = async (): Promise<void> => {
        const response = await fetch('/foo', {
            method: 'POST',
            body: canvas.toDataURL()
        });
    }

	$: if(canvas) {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        const context = canvas.getContext('2d');
        context.lineWidth = settings.radius * 2;
    }
</script>

<div class="canvas-actions">
    <button type="button" on:click={clear}>{settings.clearButtonText}</button>
    <button type="button" on:click={undo}>{settings.undoButtonText}</button>
    <button type="button" on:click={redo}>{settings.redoButtonText}</button>
    <button type="button" on:click={save}>{settings.saveButtonText}</button>
</div>
<div class="canvas-container">
    <canvas bind:this={canvas} width={settings.width * settings.scale} height={settings.height * settings.scale}
        on:mousedown={startDrawing}
        on:mousemove={draw}
        on:mouseup={stopDrawing}
    />
</div>