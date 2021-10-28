---
lastmod: Sun Nov 21 14:40:44 2010
links: {}
stylesheets: []
tags:
- script
- php
- lighttpd
- webserver
- image
- generate
- blog
title: Deep Linking images from foreign domains
---
[[!meta title="Deep Linking images from foreign domains" ]]

# Deep Linking images from foreign domains

{{< toc level="3" startlevel="2" >}}

Last week I found in my webstatistics, that I had some spike in the traffic. 

A foreign weblog or forum in hungarian language had one of the pictures on my site linked directly into their comments.

I have no Problems with people using my pictures on their Website, even less if its not a workfrom me, but from someone else, and I also only have a copy.

But what buggs me, is that other ones are reading forums and are seeing images on a foreign website and noone notices, that the image is delivered from my webserver, while on the other hand, my connection is filled up with 3000 requests in 2hours.

## The theory

To limit this behaviour for the next time, I did some configuration of the webserver(lighttpd) and some php-scripting.

In future I'll decide for every request to an image if the referrer is either empty or from my website (at least **lzer.net**). If thats the case, then normally either someone is reading my site, or he got a direct link to an article or an image. Thats fine with me.
But if  an image is embedded from another website, then the browser must have send (at least in 95% of the cases, geeks don't count) an referrer thats completely different.

In this case I'll be redirecting them to another version of the image that will have the prefix '/img/cached' in front of the original image path.


## The Webserver

Here is the part of the lighty-config reliable for that:

    $HTTP["url"] =~ "^/wiki/blog/.*\.(gif|png|jpg|jpeg)$" {
        $HTTP["referer"] !~ "^($|http://.*\.?lzer\.net)" {
            url.redirect += ( "^/(.*)$" => "/img/cached/$1"  )
            accesslog.filename = "/var/log/lighttpd/suspected_imagetheft.log"
            }
        }


These images will contain a low-quality version of the original image, and have a text written over the complete image, that the image is linked from my site, and the author should rather copy the image to his site, instead of burning my bandwidth for nothing.

The funny part is, that normally these images don't exist, before they are requested the first time.
They are created by an script that is run on the 404-Handler for this directory:

    $HTTP["url"] =~ "^/img/cached/" {
            server.error-handler-404 = "/imagegenerator/"
            }

So what's happening there?

## The Script

I had some time and fun with quick&dirty coding style. Following a commented version of "/imaggenerator/index.php":

    <?php

    $OriginalRequest=$_SERVER["REQUEST_URI"];

    $URIParts=explode('/',$OriginalRequest);
    list($EMPTY,$IMG,$CACHED,$RESTURI)=explode('/',$OriginalRequest);

First we're splitting the original request, and look, if we're really responsible. (Maybe someone directly called the script with a bad intention?)

    if ( ('' == $EMPTY) and ('img' == $IMG)  and ('cached' == $CACHED) ) {

If that's fine, then we're splitting the request and build up the filenames of the original file to load and where the new file will go to

	$I=3;
	$RealImgFN='/var/www';
	$NewImgFN='/var/www/img/cached';
	$NewImgDir='/var/www/img/cached';
	while($URIParts[$I]!='' ) {
		$RealImgFN.='/'.$URIParts[$I];
		$NewImgFN.='/'.$URIParts[$I];
		if (  !(($NewImgFN[strlen($NewImgFN)-4] == '.' ) or ($NewImgFN[strlen($NewImgFN)-5] == '.' )))	{ $NewImgDir.='/'.$URIParts[$I]; }		
		$I++;
		}

If we have that, we check for the filetype and load the file for the first time.
		
	switch(strtolower(substr($RealImgFN,-4,4))) {
			case ".jpg" : 
				$im     = imagecreatefromjpeg($RealImgFN);
				break;
			case ".png" : 
				$im     = imagecreatefrompng($RealImgFN);
				imagesavealpha($im,true);
				$is_png=true;
				break;
			case ".gif" : 
				var_dump("GIF");
				$im     = imagecreatefromgif($RealImgFN);
				break;
		}		
	if (strtolower(substr($RealImgFN,-5,5))==".jpeg") {
		$im= imagecreatefromjpeg($RealImgFN);
		}

If it's an PNG-File, we remember that.  At least we should now have "$im" loaded. If thats not the case, then we redirect to a generic image (look down)

	if ($im) {
			@mkdir($NewImgDir,0777,true);			

Why mode 0777 ? Because the umask will take care of the maximal rights on my webserver. You would have to check for that yourself.
			if (!is_png) {
				imagejpeg($im,$NewImgFN.".tmp.jpg",5);
				$im=imagecreatefromjpeg($NewImgFN.".tmp.jpg");
				}

But here's the fun part: If it's not an PNG (Which will probably have Alpha(Transparency)-Information in it, which get lost on JPEG-Compression) then we first write it out as an temp-image with JPEG-Compression 5 (Which is VERY LOW quality) and reopen it.

Then we prepare for the text to write into the image.

			$string1	= "Image linked to http://wa.lzer.net";
			$fontfile='/usr/share/fonts/truetype/ttf-dejavu/DejaVuSans-Bold.ttf';
			$blue = imagecolorallocate($im, 64, 144, 255);
			$black = imagecolorexactalpha($im, 0, 0, 0,64);
			$red = imagecolorallocate($im, 224, 64, 32);
			$white = imagecolorexactalpha($im, 255, 255,255,64);
	
			$maxX=imagesx($im);
			$maxY=imagesy($im);

And here it becomes interesting:

I want to write 2 Lines diagonally over the image, in a good, readable size. This means I'll have to calculate the angle first, in which to write the text. As it happens, thats the arcustangens of the image dimensions.
			
			$ratio=$maxX/$maxY;
			$mean=atan2($maxY,$maxX);
			$ang=rad2deg($mean);
			
Then I need to find out, which fontsize to select, so that its still readable. I increase the fontsize by one and look of the size of the boundingbox of the text:
			
			$bbx=1;$bby=1;$fs=1;
			while ((abs(($maxX/$bbx)*($maxY/$bby))>3) or (fs>120)) {			
				$fs+=1;
				$BBox=imagettfbbox($fs,$ang,$fontfile,$string1);
				$bbx=$BBox[4]-$BBox[0];
				$bby=$BBox[5]-$BBox[1];
				$e=abs(($maxX/$bbx)*($maxY/$bby));
				}

Because I write two lines, I'm moving the text a bit away from the diagonale

			$px=($maxX-$bbx)/2- (1*sin(deg2rad($ang))*$fs);
			$py=($maxY-$bby)/2- (1*cos(deg2rad($ang))*$fs);
			$ts=$fs;
			
I told you, it's quick 'n dirty, didn't I? So for the second Textline I store the main variables away,
and do the position calculation again. 

			$BBox1=$BBox;
			$px1=$px;
			$py1=$py;
			$ts1=$ts;
			
			$string="to the Author: Please copy the original to your site!";
			$fs*=0.6;
			$BBox=imagettfbbox($fs,$ang,$fontfile,$string);
			$bbx=$BBox[4]-$BBox[0];
			$bby=$BBox[5]-$BBox[1];
			$px=($maxX-$bbx)/2+ (2*sin(deg2rad($ang))*$fs);
			$py=($maxY-$bby)/2+ (2*cos(deg2rad($ang))*$fs);
			$ts=$fs;
			

Now comes another funny part:

Because Sometimes the text is hard to read on colored Images, I wanted the background of the text come grey. I'm trying to create a gradient of the boundingbox of the text to the outside of the image. It's ugly code, I dare to comment ...


			$points=array (
							$px+$BBox[6] , $py+$BBox[7],
							$px+$BBox[0] , $py+$BBox[1],
							$px+$BBox[2] , $py+$BBox[3],
							$px+$BBox[4] , $py+$BBox[5],
							$px1+$BBox1[2],$py1+$BBox1[3],
							$px1+$BBox1[4],$py1+$BBox1[5],
							$px1+$BBox1[6],$py1+$BBox1[7],
							$px1+$BBox1[0],$py1+$BBox1[1]
					);
					
			$p1dx=0*imagesx($im);$p1dy=1*imagesy($im);
			$p2dx=0*imagesx($im); $p2dy=1*imagesy($im);
			$p3dx=1*imagesx($im); $p3dy=1*imagesy($im);
			$p4dx=1*imagesx($im); $p4dy=0.5*imagesy($im);
			$p5dx=1*imagesx($im); $p5dy=0.5*imagesy($im);
			$p6dx=1*imagesx($im); $p6dy=0*imagesy($im);
			$p7dx=0*imagesx($im); $p7dy=0*imagesy($im);
			$p8dx=0*imagesx($im); $p8dy=0.5*imagesy($im);

			$AlphaGrey=imagecolorexactalpha($im,127,127,127,96);
			
			$delta=(
					abs($p1dx-$points[0])+abs($p1dy-$points[1])+
					abs($p2dx-$points[2])+abs($p2dy-$points[3])+
					abs($p3dx-$points[4])+abs($p3dy-$points[5])+
					abs($p4dx-$points[6])+abs($p4dy-$points[7])+
					abs($p5dx-$points[8])+abs($p5dy-$points[9])+
					abs($p6dx-$points[10])+abs($p6dy-$points[11])+
					abs($p7dx-$points[12])+abs($p7dy-$points[13])+
					abs($p8dx-$points[14])+abs($p8dy-$points[15])
				);
			
			$f=0.75;
			$c=0;
			while (($delta>128) and ($c++ < 200))
				{
				imagefilledpolygon($im,$points,8,$AlphaGrey);
				
				$points[0]=(($points[0]*$f)+((1-$f)*$p1dx));$points[1]=(($points[1]*$f)+((1-$f)*$p1dy));
				$points[2]=(($points[2]*$f)+((1-$f)*$p2dx));$points[3]=(($points[3]*$f)+((1-$f)*$p2dy));
				$points[4]=(($points[4]*$f)+((1-$f)*$p3dx));$points[5]=(($points[5]*$f)+((1-$f)*$p3dy));
				$points[6]=(($points[6]*$f)+((1-$f)*$p4dx));$points[7]=(($points[7]*$f)+((1-$f)*$p4dy));
				$points[8]=(($points[8]*$f)+((1-$f)*$p5dx));$points[9]=(($points[9]*$f)+((1-$f)*$p5dy));
				$points[10]=(($points[10]*$f)+((1-$f)*$p6dx));$points[11]=(($points[11]*$f)+((1-$f)*$p6dy));
				$points[12]=(($points[12]*$f)+((1-$f)*$p7dx));$points[13]=(($points[13]*$f)+((1-$f)*$p7dy));
				$points[14]=(($points[14]*$f)+((1-$f)*$p8dx));$points[15]=(($points[15]*$f)+((1-$f)*$p8dy));
				
				$delta=(
						abs($p1dx-$points[0])+abs($p1dy-$points[1])+
						abs($p2dx-$points[2])+abs($p2dy-$points[3])+
						abs($p3dx-$points[4])+abs($p3dy-$points[5])+
						abs($p4dx-$points[6])+abs($p4dy-$points[7])+
						abs($p5dx-$points[8])+abs($p5dy-$points[9])+
						abs($p6dx-$points[10])+abs($p6dy-$points[11])+
						abs($p7dx-$points[12])+abs($p7dy-$points[13])+
						abs($p8dx-$points[14])+abs($p8dy-$points[15])
					);
				}
			

But when that's ready, then we finally write the text itself. Having an positive offset in white and an negative offset in black gives an additional effect, and should make it even more readable.

			imagefttext($im,$ts1,$ang,$px1-1,$py1-1,$white,$fontfile,$string1);
			imagefttext($im,$ts1,$ang,$px1-0,$py1-1,$white,$fontfile,$string1);
			imagefttext($im,$ts1,$ang,$px1-1,$py1-0,$white,$fontfile,$string1);
			imagefttext($im,$ts1,$ang,$px1-2,$py1-2,$white,$fontfile,$string1);
			imagefttext($im,$ts1,$ang,$px1+1,$py1+1,$black,$fontfile,$string1);
			imagefttext($im,$ts1,$ang,$px1+0,$py1+1,$black,$fontfile,$string1);
			imagefttext($im,$ts1,$ang,$px1+1,$py1+0,$black,$fontfile,$string1);
			imagefttext($im,$ts1,$ang,$px1+2,$py1+2,$black,$fontfile,$string1);
			imagefttext($im,$ts1,$ang,$px1,$py1,$blue,$fontfile,$string1);

			imagefttext($im,$ts,$ang,$px-1,$py-1,$white,$fontfile,$string);
			imagefttext($im,$ts,$ang,$px+1,$py+1,$black,$fontfile,$string);
			imagefttext($im,$ts,$ang,$px,$py,$red,$fontfile,$string);

Now the imagecontent is ready. We´re writing the image (PNG stays PNG).
And after that, we redirect to the same location again.
			
			@mkdir($NewImgDir,0777,true);
			if ($is_png) {
				imagepng($im,$NewImgFN);
				} else {
				imagejpeg($im,$NewImgFN,50);				
				}
			imagedestroy($im);

But now' the requested Image should be in its place and get directly delivered. Otherwise it'll be recreated again.

			header("Location: $OriginalRequest");
		} else {
			header("Location: /img/cached/ImageTheft.png");
		}
	}
    ?>

This script was hacked in two days and has enough potential to get optimized, but I was in the need of an quick solution. The this is, that the cached images are more compressed and therefore also spare me some bandwidth.



