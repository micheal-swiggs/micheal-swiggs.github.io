---
layout: post
title: Apply Bash command to each folder
---


Today I needed to individually delete each folder within a directory. The command I used is:

    for dir in folder/*; do (SOMECOMMAND $dir); done

For example, if I needed to see the size of each folder within a directory:

    for dir in folder/*; do (du -h --max-depth=1); done

Or if I needed to securely delete each folder within a directory:

    for dir in folder/*; do (srm -rlv $dir); done
