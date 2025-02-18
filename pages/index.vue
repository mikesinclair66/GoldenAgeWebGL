<template>
    <canvas id="webgl-canvas"></canvas>
</template>

<script>
    import '@/assets/css/main.scss';
    import basic_vertex from '@/assets/glsl/basic_vertex.glsl';
    import basic_fragment from '@/assets/glsl/basic_fragment.glsl';

    import { mat4 } from 'gl-matrix';
    import { compileShaderProgram, detectShaderProgramError } from '@/assets/scripts/compile_shader_program.js';
    import prepareRender from '@/assets/scripts/prepare_render.js';

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

            //compile basic shader
            const basicShaderProgram = compileShaderProgram(gl, basic_vertex, basic_fragment);
            detectShaderProgramError(gl, basicShaderProgram);
            gl.useProgram(basicShaderProgram);

            //initialize VBO and IBO
            const vertices = [
                -0.5,  0.5,  0.0, // top-left
                -0.5, -0.5,  0.0, // bottom-left
                0.5, -0.5,  0.0, // bottom-right
                0.5,  0.5,  0.0, // top-right
            ];

            const indices = [
                0, 1, 3, // First triangle (top-left, bottom-left, top-right)
                1, 2, 3 // Second triangle (bottom-left, bottom-right, top-right)
            ];

            const VBO = gl.createBuffer();
            gl.bindBuffer(gl.ARRAY_BUFFER, VBO);
            gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW);

            const IBO = gl.createBuffer();
            gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, IBO);
            gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, new Uint16Array(indices), gl.STATIC_DRAW);

            //Clears the scene background, generally happens per frame
            prepareRender(gl);

            //In OpenGL ES (The graphics API WebGL is based off), we need to initialize variables that
            //act as a link from here to the variables in the shader source code. That way, we can
            //assign the values of our variables here to the variables in the shader program.
            const VBOProgramLocation = gl.getAttribLocation(basicShaderProgram, 'a_vertexPosition');
            const shaderProjectionLocation = gl.getUniformLocation(basicShaderProgram, 'u_projectionMatrix');
            const shaderModelViewMatrixLocation = gl.getUniformLocation(basicShaderProgram, 'u_modelViewMatrix');

            gl.enableVertexAttribArray(VBOProgramLocation);
            {
                const EDGES = 3, TYPE = gl.FLOAT, NORMALIZE = false, STRIDE = 0, OFFSET = 0;
                gl.bindBuffer(gl.ARRAY_BUFFER, VBO);
                gl.vertexAttribPointer(
                    VBOProgramLocation, //variable that stores the shader attribute variable a_vertexPosition
                    EDGES, //number of edges in the shape
                    TYPE, //type of the values used in creating the VBO
                    NORMALIZE, //not important for now
                    STRIDE, //also not yet
                    OFFSET); //will explain later
            }

            const FOV = 45 * Math.PI / 180;   //field of view in radians
            const ASPECT_RATIO = gl.canvas.width / gl.canvas.height;
            const ZNEAR = 0;
            const ZFAR = 100.0;

            const projectionMatrix = mat4.create();
            mat4.perspective(projectionMatrix,
                FOV,
                ASPECT_RATIO,
                ZNEAR,
                ZFAR);
            gl.uniformMatrix4fv(
                shaderProjectionLocation,
                false,
                projectionMatrix);

            const modelViewMatrix = mat4.create();
            mat4.translate(modelViewMatrix, modelViewMatrix, [-0.0, 0.0, -6.0]);
            gl.uniformMatrix4fv(
                shaderModelViewMatrixLocation,
                false,
                modelViewMatrix);

            {
                const TYPE = gl.TRIANGLES;
                gl.drawElements(TYPE,
                    indices.length,
                    gl.UNSIGNED_SHORT,
                    0);
            }
        }
    }
</script>