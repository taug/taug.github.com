---
layout: post
title: "Cocos2dx资源搜索路径"
description: ""
category: cocos2dx
tags: [cocos2dx,serch path]
---
{% include JB/setup %}

一直有一个疑惑，cocos2dx是怎么搜索资源路径的？看了源代码之后终于弄明白了，写下来以免忘记。

##添加搜索路径
    FileUtils::getInstance()->addSearchPath("src");
    FileUtils::getInstance()->addSearchResolutionsOrder("res");

添加的搜索路径会被在前面添加一个根目录，保存起来，也可以填添加一个绝对目录。
添加的分辨率目录也会被直接保存起来。

    std::vector<std::string> _searchPathArray;
    std::vector<std::string> _searchResolutionsOrderArray;

这两个变量就是用来保存设置的搜索路径的。

##搜索文件
FileUtils会根据在默认的路径和添加的搜索路径里面搜索资源文件，搜索方法的源码如下：

    std::string FileUtils::fullPathForFilename(const std::string &filename)
    {
        if (filename.empty())
        {
            return "";
        }

        if (isAbsolutePath(filename))
        {
            return filename;
        }

        // Already Cached ?
        auto cacheIter = _fullPathCache.find(filename);
        if( cacheIter != _fullPathCache.end() )
        {
            return cacheIter->second;
        }

        // Get the new file name.
        const std::string newFilename( getNewFilename(filename) );

        std::string fullpath;

        for (auto searchIt = _searchPathArray.cbegin(); searchIt != _searchPathArray.cend(); ++searchIt)
        {
            for (auto resolutionIt = _searchResolutionsOrderArray.cbegin(); resolutionIt != _searchResolutionsOrderArray.cend(); ++resolutionIt)
            {
                fullpath = this->getPathForFilename(newFilename, *resolutionIt, *searchIt);

                if (fullpath.length() > 0)
                {
                    // Using the filename passed in as key.
                    _fullPathCache.insert(std::make_pair(filename, fullpath));
                    return fullpath;
                }
            }
        }

        CCLOG("cocos2d: fullPathForFilename: No file found at %s. Possible missing file.", filename.c_str());

        // XXX: Should it return nullptr ? or an empty string ?
        // The file wasn't found, return the file name passed in.
        return filename;
    }

如果我们要找一个文件bg.jpg，调用下面的代码：

    FileUtils::getInstance():fullPathForFilename("bg.jpg");

Fileutils会在默认的目录及我们上面添加的目录里面搜索,直到搜索到为止。默认的路径会先搜索到，我们添加的索搜路径会按顺序搜索到。
所以搜索到的路径顺序是：

    defaule_path/bg.jpg
    defaule_path/res/bg.jpg
    defaule_path/src/bg.jpg
    defaule_path/src/res/bg.jpg

搜索到了就会返回这个文件的全路径，如果没有搜索到就返回原文件名。知道了这些我们就可以很好地设置搜索路径，比如把通用的资源放在资源
根目录下，把特定分辨率的资源放在特定的目录下，这样资源管理起来就很清晰了。

