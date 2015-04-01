# gulp-cmd

***
> seajs(CMD) Module transform and concat pulgin for gulp

## Install

```
$ npm install --save-dev gulp-cmd
```

注：最好的模式是每个文件名都是不同的，因为所有模块都是基于文件名来分配id，对于其他不同文件夹下的同名文件，为了避免冲突，命名方式会被更换成是{文件名}_gulp-cmd_{uid}，对于不同文件夹下的同名文件名，需要注意一点的是ignore中需要用精确匹配

## Usage

```
var gulp = require( 'gulp' ),
    cmd = require( 'gulp-cmd' );
    
gulp.task( 'cmd', function(){
    return gulp.src( 'src/js/main.js' )
        .pipe( cmd() )
        .pipe( gulp.task('build/js') );
}); 
```

## API

### cmd( options )

Unsupported files are ignored.

### options

#### encoding 

Type : `String`

Default : `utf-8`

#### ignore

Type : `Array`

Ignored module list. e.g. combo module `main`, want ignore dependencies `global` and `common`, configuration ignore list :

```
ignore : [ 'global', 'common' ]
```

模糊匹配：不带路径信息

ignore : ['a'] 会匹配 src/a src/b/a

精确匹配：带路径信息为带/

ignore : ['src/a'] 会匹配 src/a 但不会匹配 src/b/a

The ignore configuration must is name of the module. 

#### map

When use `seajs.use`, module id is `foo/bar/biz`, but the module file with respect to `gulp.src` path is `./biz.js`, use `map` configuration.

```
map : {
    'foo/bar/biz' : './biz'
}
```

#### plugins

`plugins` for special module. A module need combo Handlebars tpl module, but tpl will compile before combo.

```
var handlebars = require( 'gulp-handlebars' ),
      wrap = require( 'gulp-wrap' );
      
...
plugins : [{
    ext : [ '.tpl' ],
    use : [{
            plugin : handlebars, 
        },{
            plugin : wrap,
            param : ['define(function(){return Handlebars.template(<%= contents %>)});']
    }]
}]
```

## concat rule

Module `a.js` :

```
define(function(){
    var b = require( 'deps/b' );
    return 'a' + ' ' + b;
});
```

Module `b.js` :

```
define(function(){
    return 'b';
});
```

gulp code :

```
gulp.src( 'src/a.js' )
    .pipe( cmd() )
    ...
```

Combo after `a.js` :

```
define('b',function(){
    return 'b';
});
define('a',['b'],function(){
    var b = require( 'b' );
    return 'a' + ' ' + b;
});
```

File `main.js` :

```
seajs.use( 'a' );
```

Gulp code : 

```
gulp.src( 'src/main.js' )
    .pipe( cmd() )
    ...
```

Combo after `main.js` :

```
define('b',function(){
    return 'b';
});
define('a',['b'],function(){
    var b = require( 'b' );
    return 'a' + ' ' + b;
});
seajs.use( 'a' );
```

## Parse `seajs.config`

只有在有seajs.use的时候才有用

`gulp-cmd` will parse `alias` `vars` `paths` in `seajs.config`, other configuration is ignored, the configuration value must be a `String`, will ignored variable. see more [seajs.config](https://github.com/seajs/seajs/issues/262). [test/src/m.js](https://github.com/elover/gulp-cmd/blob/master/test/src/m.js) and [test/build/m.js](https://github.com/elover/gulp-cmd/blob/master/test/build/m.js) is parse example.

## License

MIT @ [nan wei](https://github.com/elover)