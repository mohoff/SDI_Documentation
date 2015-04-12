
******
SPHINX
******

.. comments

Formatting
##########

Sphinx is flexible in terms of the marking of heading and sub(sub)headings. You can define your own style like using and reusing ==== ********** ++++++ for instance.

Subheadline
***********

*italic*

**bold**

``inline code``

::

  multiline code. needs to be indented and wrapped with 2 blank lines

.. code-block:: html
  :linenos:

  codeblock examples
  apparently with line numbers with option :linenos:
  sweet
  4
  5
  6

Listing (Numbered)
##################
1. bla
2. blup
#. blopp

Listing (Unordered)
###################
* bullet 1
* bullet 2
* bullet 3

Image
#####
.. image:: ./images/goik.jpg
    :align: center
    :alt: alternate text

options von image-derivates:

* :height: 100px
* :width: 100px
* :align: center
* :alt: alternate text

Gray Box
########
.. topic:: Hint box

    sexy gray box

Red Warning Box
###############
.. warning::

   Never, ever, ignore this warning!

Sidebar Box
###########
.. sidebar:: Sidebar Hint
  :subtitle: Optional Sidebar Subtitle

  Subsequent indented lines comprise
  the body of the sidebar, and are
  interpreted as body elements.

  sidebar appears as floating box and may not appear nicely.

Math
####
.. warning::

    Somehow, math expressions aren't displayed correctly!

Ahhh. lateX needs to be installed so equations can get rendered into PNG

:math:`\alpha > \beta`

.. math::

    n_{\mathrm{offset}} = \sum_{k=0}^{N-1} s_k n_k

Usage of a Footnote
###################
This text is quoted so it needs a footnote [#f1]_ .

Internal Reference
##################
Click here `Image`_

All headings and subheadings are considered as internal hyperlinks

External Reference
##################
Follow this link to get to `hdm-stuttgart.de <http://www.hdm-stuttgart.de/>`_

Glossary
########
.. glossary::

    paul
        study colleague
    goik
        lecturer


.. rubric:: Footnotes

.. [#f1] Text of the first footnote.
