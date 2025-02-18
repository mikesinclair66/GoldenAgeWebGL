<template>
    <canvas id="webgl-canvas"></canvas>
</template>

<script>
    import '@/assets/css/main.scss';
    import { mat4 } from 'gl-matrix';

    export default {
        mounted(){
            const canvas = document.getElementById("webgl-canvas");
            if(!canvas){
                console.log("An error occurred loading the canvas.");
                return;
            }

            const width = window.innerWidth;
            const height = window.innerHeight;
            const devicePixelRatio = window.devicePixelRatio || 1;

            //drawing dimensions
            canvas.width = width * devicePixelRatio;
            canvas.height = height * devicePixelRatio;
            
            //display size
            canvas.style.width = `${width}px`;
            canvas.style.height = `${height}px`;

            const gl = canvas.getContext("webgl");
            if(!gl){
                console.log("An error occurred initializing webgl");
                return;
            }

            function compileShader(gl, shaderType, shaderKeyword, shaderSrc){
                const shader = gl.createShader(shaderType);
                if(!shader){
                    console.log(`An error occurred creating the ${shaderKeyword} shader.`);
                    return null;
                }

                gl.shaderSource(shader, shaderSrc);
                gl.compileShader(shader);

                if(!gl.getShaderParameter(shader, gl.COMPILE_STATUS)){
                    console.log(`An error occured while compiling the ${shaderKeyword}:\n${gl.getShaderInfoLog(shader)}`);
                    gl.deleteShader(shader);
                    return null;
                }

                return shader;
            }

            //vertex shader
            const vertexShader = compileShader(gl, gl.VERTEX_SHADER, "vertex", `
                attribute vec4 aVertexPosition;
                uniform mat4 uModelViewMatrix;
                uniform mat4 uProjectionMatrix;
                void main() {
                    gl_Position = uProjectionMatrix * uModelViewMatrix * aVertexPosition;
                }
            `);

            //fragment shader
            const fragmentShader = compileShader(gl, gl.FRAGMENT_SHADER, "fragment", `
                void main() {
                    gl_FragColor = vec4(1.0, 1.0, 1.0, 1.0); // White
                }
            `);

            //create shader program
            const shaderProgram = gl.createProgram();
            gl.attachShader(shaderProgram, vertexShader);
            gl.attachShader(shaderProgram, fragmentShader);
            gl.linkProgram(shaderProgram);

            if (!gl.getProgramParameter(shaderProgram, gl.LINK_STATUS)) {
                alert('Unable to initialize the shader program: ' + gl.getProgramInfoLog(shaderProgram));
                return null;
            }

            const programInfo = {
                program: shaderProgram,
                attribLocations: {
                    vertexPosition: gl.getAttribLocation(shaderProgram, 'aVertexPosition'),
                },
                uniformLocations: {
                    projectionMatrix: gl.getUniformLocation(shaderProgram, 'uProjectionMatrix'),
                    modelViewMatrix: gl.getUniformLocation(shaderProgram, 'uModelViewMatrix'),
                },
            };

            //initliaze buffer
            const positionBuffer = gl.createBuffer();
            gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);
            gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([
                -0.5, -0.5, 0.0,
                0.5, -0.5, 0.0,
                0.0,  0.5, 0.0
            ]), gl.STATIC_DRAW);

            const buffer = {
                position: positionBuffer
            };

            // Draw the scene
            gl.clearColor(0.0, 0.0, 0.0, 1.0);
            gl.clearDepth(1.0);
            gl.enable(gl.DEPTH_TEST);
            gl.depthFunc(gl.LEQUAL);

            gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);

            const fieldOfView = 45 * Math.PI / 180;   // in radians
            const aspect = gl.canvas.width / gl.canvas.height;
            const zNear = 0.1;
            const zFar = 100.0;
            const projectionMatrix = mat4.create();

            mat4.perspective(projectionMatrix,
                fieldOfView,
                aspect,
                zNear,
                zFar);

            const modelViewMatrix = mat4.create();

            mat4.translate(modelViewMatrix, modelViewMatrix, [-0.0, 0.0, -6.0]);

            // Tell WebGL how to pull out the positions from the position buffer into the vertexPosition attribute
            const numComponents = 3;
            const type = gl.FLOAT;
            const normalize = false;
            const stride = 0;
            gl.bindBuffer(gl.ARRAY_BUFFER, buffer.position);
            gl.vertexAttribPointer(
                programInfo.attribLocations.vertexPosition,
                numComponents,
                type,
                normalize,
                stride,
                0);//offset
            gl.enableVertexAttribArray(programInfo.attribLocations.vertexPosition);

            gl.useProgram(programInfo.program);

            gl.uniformMatrix4fv(
                programInfo.uniformLocations.projectionMatrix,
                false,
                projectionMatrix);
            gl.uniformMatrix4fv(
                programInfo.uniformLocations.modelViewMatrix,
                false,
                modelViewMatrix);
            const vertexCount = 3;
            gl.drawArrays(gl.TRIANGLES, 0, vertexCount);
        }
    }
</script>