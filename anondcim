#!/bin/sh -e

function die() { 
    echo -e "$*" >&2
    exit 1
}

function r() {
    echo $(($RANDOM % $1))
}

max=$#
cur=1

[ $max -gt 0 ] || die "Usage:\n[IMG_PREFIX=your_prefix] $0 img1.jpg img2.jpg ..."
[ -x "$(which convert)" ] || die "ImageMagick is not installed"
[ -x "$(which jhead)" ] || die "jhead is not installed"

while [ $# -gt 0 ]; do
    dst="$IMG_PREFIX$(seq -w $cur $max|head -n1).jpg"
    echo -e "[$((100*cur/max))%]\t$1\t-> $dst"

    [ -f "$1" ] || die "$1 does not exist"
    (! [ -e "$dst" ]) || die "$dst already exists"

    read W H <<EOF
        $(identify $1 |cut -f3 -d\ |tr x \ )
EOF

    [ $W -ge 100 ] && [ $H -ge 100 ] || die "image is too small"
    
    if [ $W -ge 1000 ]; then DW=$((W / 100)); else DW=10; fi
    if [ $H -ge 1000 ]; then DH=$((H / 100)); else DH=10; fi
    
    W=$(($W-1))
    H=$(($H-1))
    
    convert $1 \
	-colorspace RGB \
	-distort Perspective "$(
        (   echo $(r $DW)           $(r $DH)           0  0
	    echo $(($W - $(r $DW))) $(r $DH)           $W 0
	    echo $(r $DW)           $(($H - $(r $DH))) 0  $H
	    echo $(($W - $(r $DW))) $(($H - $(r $DH))) $W $H
        ) | tr " \n" ", ")" \
	-filter gaussian -define filter:support=5 -define filter:sigma=0.5 \
	-attenuate 2 +noise Uniform \
	-resize 50% \
	-colorspace sRGB \
	"$dst"

    jhead -purejpg -q "$dst" || die "removing meta-data failed"

    cur=$(($cur + 1))
    shift
done
