---
layout: post
title: "使用Text Kit来绘制和管理文本"
date: 2016-11-04 08:29:59 +0800
categories:
---

引言：本文来自Apple官方文档《Text Programming Guide for iOS: Using Text Kit to Draw and Manager》，可能会由于本人英语水平较低（未过英语四级考试）、理解能力和文档时效等问题造成与原文出入较大，阅读时请慎重！如遇到问题，请在评论中指出。

正如《Displaying Text Content in iOS》部分描述，UIKit 框架中提供了一些用来在App的用户界面中展示文本的类：UITextView, UITextField, UILabel 和 UIWebView。由 UITextView 类创建的文本视图意在用来展示大量的文本。UITextView 底层有一个非常强大的布局引擎叫做 Text Kit。如果你需要自定义或者介入这个布局过程，你可以使用 Text Kit。对于更细粒度的文本和特殊的需要自定义解决方案的需求，可以使用《Lower Level Text-handling Technologies》中描述的更底层的技术作为替代。

TextKit 是 UIKit 框架中提供的高品质的保存、布局、展示精细排版的所有特征（如字距、连字、换行和对齐）的排版服务的类和协议的集合。TextKit 构建于 CoreText 之上，所以它提供了同样的速度与强大。UITextView 完全集成了 Text Kit，它提供的编辑和展示功能允许用户输入文本、指定格式属性并查看结果。Text Kit 其他方法提供了文本存储和布局功能。下图展示了 Text Kit 在 iOS 文本和图像框架中的位置。

![Text Kit Framework Position]({{ site.baseurl }}/images{{ page.id }}/1.png)

Text Kit 给了你在用户界面元素上渲染文本的完整的控制。除 UITextView 外，UITextField 和 UILabel 也是在 Text Kit 之上构建的，它可以无缝地和动画、UICollectionView 和 UITableView 集成。Text Kit 被设计为拥有完全可扩展的面向对象架构，支持子类化、委托和一组完整的通知，支持深度定制。

## TextKit主要的成员对象

Text Kit 主要对象之间的数据流图如下图。Text views 是 UITextView 类的实例，Text containers 是 NSTextContainer类的实例，layout manager 是 NSLayoutManager 类的实例，Text storage 是 NSTextStorage 类的实例。在 Text Kit 中，一个 NSTextStorage 对象存储用于 UITextView 对象显示和 NSLayoutManager 对象在一个 NSTextContainer 对象上布局的文本。

![Primary Text Kit Objects]({{ site.baseurl }}/images{{ page.id }}/2.png)

一个 NSTextContainer 对象可以定义一个可供文本布局的区域。text container 通常会定义一个方形区域，但可以通过创建一个 NSTextContainer 的子类来定义其他形状如圆、五边形或不规则的形状。text container 不仅定义了可以填充文本区域的边界，同时它还维护了一个定义区域中不能布局文本的地方（例外区）的 Bezier paths 数组。当布局发生时，文本在例外区以外布局，这就为包含图像和其他非文本元素提供了一种方式。

NSTextStorage 为 Text Kit 的扩展文本处理系统（extended text-handling system）定义了基本的存储机制。NSTextStorage 是 NSMutableAttributedString 的一个子类，储存了文本系统操作的文本和属性。它保证了文本和属性可以在编辑操作中保持一贯的状态。除了保存文本，NSTextStorage 对象还管理了一些 NSLayoutManager 对象的集合，在必要时将文本或属性的变化通知给它们来重新布局或重新展示。

NSLayoutManager 对象统筹其他文本处理对象的操作。协调将 NSTextStorage 中的数据转换为在视图的显示区域中渲染的文本的操作。将 Unicode 码映射为字符并监督 NSTextContainer 对象定义的区域上的布局。

> 注意： NSLayoutManager、NSTextStorage 和 NSTextContainer 可以在子线程中访问，但要保证只在单一的线程中被访问。

## 文本属性

Text Kit 处理三种文本属性：字符属性、段落属性和文档属性。字符属性包括如字体、颜色和下标等可以在单独的字符或一个区间的字符关联的特性。段落属性是如缩进、tabs和行间距等的特性。文档属性包括如纸张大小，边距和视图缩放比例等文档级别的特性。

### 字符属性

属性字符串（attributed string）用字典中的键值对来存储字符属性。键作为属性的名称，用一个字符串常量标识符（如NSFontAttributeName）来表示。下图展示了一个属性字典作用于一个属性字符串中的一个区间上。

![Composition of an attributed string]({{ site.baseurl }}/images{{ page.id }}/3.png)

理论上，属性字符串中每个字符都有一个相关联的属性字典。然而通常一个属性字典作用于更长的字符区间上。NSAttributedString 类提供了根据字符的位序获取关联的属性字典和该属性字典的属性值作用的区间的方法如：`attributesAtIndex:effectiveRange:`.

除了使用预定义的属性外，你可以选择向一个字符区间指定任意的属性键值对。你可以向 NSTextStorage 中合适的字符区间通过 NSMutableAttributedString 的`addAttribute:value:range:`方法添加属性，你也可以创建一个包含一些自定义属性的键和值的字典然后通过`addAttributes:range:`方法一步将它们添加到一个字符区间上。要使用自定义的属性时，你需要自定义一个的 NSLayoutManager 的子类来使用它们。你自定义的子类应该重写`drawGlyphsForGlyphRange:atPoint:`方法。在这个方法里可以先调用父类相应的方法来绘制字符区间然后在上面加上你自定义的属性，或者也完全使用自己的方式绘制字符。

### 段落属性

段落属性影响布局管理器（layout manager）将文本行布置到页面航的段落中的方式。文本系统在 NSParagraphStyle 类的对象中封装段落属性。预定义的字符属性中的 NSParagraphStyleAttributeName 键的值指向一个包含了这个字符区间的段落属性的 NSParagraphStyle 对象。属性修复（attribute fixing）确保只有一个 NSParagraphStyle 对象属于各个段落中的字符。

段落属性包含如对其、制表符停止（tab stop）、换行模式、和行间距（也被称为leading）等特性。

### 文档属性

文档属性属于文档整体。文档属性包含纸张大小、边距和视图缩放比例等特性。尽管文本系统中没有内置的存储文档属性的机制，NSAttributedString 的初始化方法如`initWithRTF:documentAttributes:`可以将从 RTF 或 HTML 数据流中派生出的文档属性填充到你提供的占位字典对象中。相反的，如果给写入RTF数据的方法如`RTFFromRange:documentAttributes:`传递一个包含文档属性的字典，它会写入文档属性。

### 属性修正

编辑属性字符串可能引起的前后不一致必须通过属性修复（attribute fixing）来清除。UIKit 通过定义`fixAttributesInRange:`方法扩展了 NSMutableAttributedString 来修复附件、字符和段落属性中的矛盾。这些方法保证了当附件字符被删除时其所关联的属性也将不复存在，字符属性只应用在支持这个字体的字符上，段落属性在整个段落中保持一致。

## 使用编程的方式改变文本存储

NSTextStorage 对象为 Text Kit 提供字符仓库的服务。此数据的格式是字符序列（使用unicode编码）和关联了属性（如字体、颜色和段落属性）的属性字符串。表示属性字符串的类有 NSAttributedString 和 NSMutableAttributedString（NSTextStorage 就是它的子类）。如字符属性小节描述，每一个文本块中的字符都有一个键值对字典与其相关联。键为属性名（如NSFontAttributeName），对应的值指定字符的这个属性的特征（如Helvetica 12-point）。

通过编程的方式编辑text storage对象有三个阶段。第一个阶段是向其发送一个`beginEditing`消息来宣布一组修改。

在第二个阶段中，你向它发送一些编辑消息如`replaceCharactersInRange:withString:`和`setAttributes:range:`来使字符或属性改变。每次发送这样的消息时，text storage 对象调用`edited:range:changeInLength:`来跟踪从收到`beginEditing`消息后受影响的字符区间。

在第三个阶段中，当你完成了对 text storage 对象的修改，向其发送一个`endEditing`消息。这会使其调用其 delegate 的`textStorage:willProcessEditing:range:changeInLength:`和其自身的`processEditing`方法，修复记录的有修改的字符区间上的属性。

修复完属性后，text storage 对象向其 delegate 发送`textStorage:didProcessEditing:range:changeInLength:`方法，给 delegate 一个验证并进行可能的修改的时机。（尽管 delegate 可以在这个方法中修改 text storage 对象的字符属性，但不可以在 text storage 对象处于冲突状态时修改字符本身。）最后，text storage 对象向每个关联的 layout manager 发送`processEditingForTextStorage:edited:range:changeInLength:invalidateRange:`消息--指出 text storage 对象中改变了的字符区间以及这些修改的性质。layout manager 们轮流的使用这些信息来重新计算他们的文字位置并在必要时重新展示。

## 使用字体对象

计算机字体是使用了如 OpenType 或 TrueType 的格式的数据文件，包含描述一组字形的信息，以及在字形渲染中使用的各种补充信息。UIFont 类提供了获取和设置字体信息的接口。一个 UIFont 实例提供对字体特性和字形的访问。Text Kit 结合字符信息和字体信息来选择文本布局时要使用的字形。通过将字体对象传递给接收字体作为参数的方法中来使用字体对象。字体对象是不可变的，所以可以安全地在应用中多个线程中使用它们。

要使用`preferredFontForTextStyle:`或`fontWithName:size:`来创建 UIFont对象而不能使用 alloc 和 init 方法来创建。也可以用 font descriptor 通过`fontWithDescriptor:size:`来创建字体。这些方法会先从现有的字体对象中查找是否存在指定特征的字体对象，如果有就返回它，否则，根据请求的字体数据创建一个合适的字体对象。

### 文本类型

文本类型（Text styles），iOS 7 带来的由被称为动态类型(*Dynamic Type*)的机制实现的对字体的预期用途的语义描述。文本类型通过 UIFontDescriptor.h 中定义的常量（如下表所示）进行组织使用。用于由文本样式描述的目的的实际字体可能基于一些动态考虑而变化，包括由 UIApplication 属性`preferredContentSizeCategory`表示的用户的内容大小类别偏好。要获取给定文本样式的字体对象，您需要将相应的常量传递给 UIFont 方法`preferredFontForTextStyle:`. 要获取文本样式的字体描述符，请将常量传递给 UIFontDescriptor 的`preferredFontDescriptorWithTextStyle:`方法。

表：文本类型常量

---------------------------|------------------------
UIFontTextStyleHeadline    |    用于标题的字体
UIFontTextStyleSubheadline |    用于子标题的字体
UIFontTextStyleBody        |    用于正文文本的字体
UIFontTextStyleFootnote    |    用于注脚的字体
UIFontTextStyleCaption1    |    用于标准字幕的字体
UIFontTextStyleCaption2    |    用于替代字幕的字体

文本类型通过动态类型机制为应用带来了许多提高文本可读性的优势，动态类型以协调的方式响应用户的偏好，并响应辅助功能设置以增强易读性和超大类型。也就是说，当你调用`preferredFontForTextStyle:`时，返回的特定的字体除了对指定的文本类型常量用途进行处理外，还包括根据用户偏好和上下文而变化的特征，包括跟踪（字母间距）调整。

根据文本类型常量得到的字体应该被用于应用中所有的文本中，除了界面元素如按钮、标签和条（bar）等。当然你需要合适你应用的文本类型。同时，观察`UIContentSizeCategoryDidChangeNotification`也很重要，这样可以在用户修改了内容尺寸类别时重新布局。当你的应用收到这个通知时，它应该向 Auto Layout 布局的 view 发送`invalidateIntrinsicContentSize`或者向手动布局的用户界面元素发送`setNeedsLayout`消息。同时要使先前的 font 或 font descriptor 失效，然后获取新的。

### 使用字体描述

字体描述，UIFontDescriptor类的实例，提供了通过一个属性字典来描述字体的方式，并可以用它来创建 UIFont 对象。你可以从一个字体描述创建一个 UIFont 对象，也可以从 UIFont 对象得到一个描述，还可以改变这个描述然后用它生成一个新的字体对象。你也可以用一个字体描述来指定应用中提供的自定义的字体。

字体描述可以被存档，这样可以高效的使用文本类型。你不应该为文本类型缓存字体对象，因为这是动态的（根据用户的偏好不同会改变）。但你可以缓存字体的描述，然后稍后用它再创建相同特征的字体对象。

你可以使用字体描述来查询系统中符合特定属性的可用字体，然后创建符合这些属性（名称、特点、语言或其他特性）的字体对象。例如，你可用使用一个字体描述来获取所有的符合给定的字体簇名称的字体，使用CSS标准定义的簇名称：

```
UIFontDescriptor *helveticaNeueFamily = [UIFontDescriptor fontDescriptorWithFontAttributes:@{UIFontDescriptorFamilyAttribute: @“Helvetica Neue”}];
NSArray *matches = [helvetecaNeueFamily matchingFontDescriptorsWithMandatoryKeys:nil];
```

`matchingFontDescriptorsWithMandatoryKeys:`方法返回系统中所有的 Helvetica Neue 字体的描述的数组，如HelveticaNeue, HelveticaNeue-Medium, HelveticaNeue-Light, HelveticaNeue-Thin,等。

你可以通过应用字形特征（如粗体、斜体、宽松、紧凑）来修改`preferredFontForTextStyle:`方法返回的字体，可以使用字体描述来修改特定的特征：

```
UIFontDescriptor *fontDescriptor = [UIFontDescriptor preferredFontDescriptorWithTextStyle:UIFontTextStyleBody];
UIFontDescriptor *boldFontDescriptor = [fontDescriptor fontDescriptorWithSymbolicTraits:UIFontDescriptorTraitBold];
UIFont *boldFont = [UIFont fontWithDescriptor:boldFontDescriptor size:0.0];
```

这个代码片段先取到正文文本类型的字体描述，然后修改这个字体描述为加粗特征，最后使用 UIFont 类的`fontWithDescriptor:size:`方法返回一个实际用于正文文本的加粗字体对象。传递给`fontWithDescriptor:size:`方法的size为0.0是为了保护字体描述返回的原始尺寸属性。由于字体大小是由动态类型机制决定的，所以提倡这么做。


000000000000000000000000
### 激活字体特性

字体描述的另一个重要的用途就是激活和选择自体特性。字体特性是文本系统渲染文字形状时控制其外表的排版属性。字体特性只在字体设计者选择包含字体特性的字体中可用。有一些字体特性只在少许的字体中可用，而更多的特性在大多数字体中可用。另外，同一字体的不同版本，安装在不同平台的版本，它们可用的字体特性可能是不同的。

字体特性按特性类型进行分组，选择其中的一个特性就是设置对应的特性。特性类型可以是排他的也可以是不排他的，如果一个特性类型是排他的，你同时只能选择一个可用的属于该类型的特性，例如数字可以是固定宽度的或自适应宽度。如果特性类型是不排他的，你可以同时从中选择多个特性，例如连字特性，你可以选择当前字体支持的连字特性进行复合。

注意，如果为一个字体选择了不支持的特性，字体表现不会有任何变化。

有的特性是上下文相关的。这样的特性应用时会比较文字相邻位置的文字。文字系统的一个强大的地方就是布局处理能力，可以精密的处理上下文。
非上下文相关特性应用于不考虑相邻文字的情况。这些功能包括替代字形的选择将给予文字不同的外观和字形替换为目的数学排版或提高印刷复杂的。

激活Helvetica Neue Medium字体定义的两个特性类型：
NSArray *timeFeatureSettings = @[
     @{
          UIFontFeatureTypeIdentifierKey: @(kNumberSpacingType),
          UIFontFeatureSelectorIdentifierKey: @(kProportionalNumbersSelector)
     },
     @{
          UIFontFeatureTypeIdentifierKey: @(kCharacterAlertnativesType),
          UIFontFeatureSelectorIdentifierKey: @(2)
     }];
UIFont *originalFont = [NSFont fontWithName:@“HelveticaNeue-Medium” size:12.0];
UIFontDescriptor *originalDescriptor = [originalFont fontDescriptor];
UIFontDescriptor *timeDescriptor = [originalDescriptor fontDescriptorByAddingAttributes: @{UIFontDescriptorFeatureSettingsAttribute:timeFeatureSettings}];
UIFont *timeFont = [UIFont fontWithDescriptor:timeDescriptor size:12.0];
上面代码中用到的常量在Core Text框架中的SFNTLayoutTypes.h文件中定义。

由于字体特性是在字体中定义的，所以最可靠的确定字体支持的特性的方法就是直接从字体中查询。你可以使用Core Text中的CTFontCopyFeatures函数：
UIFont *font = [UIFont fontWithName:@“HelveticaNeue-Medium” size:12.0];
CFArrayRef fontFeatures = CTFontCopyFeatures((__bridge CTFontRef)font);
NSLog(@“properties = %@, fontFeatures);
＝＝＝＝＝＝打印输出＝＝＝＝＝
properties = (
        {
        CTFeatureTypeExclusive = 1;
        CTFeatureTypeIdentifier = 6;
        CTFeatureTypeName = "Number Spacing";
        CTFeatureTypeNameID = 266;
        CTFeatureTypeSelectors =         (
                        {
                CTFeatureSelectorDefault = 1;
                CTFeatureSelectorIdentifier = 0;
                CTFeatureSelectorName = "No Change";
                CTFeatureSelectorNameID = 264;
            },
                        {
                CTFeatureSelectorIdentifier = 1;
                CTFeatureSelectorName = "Proportional Numbers";
                CTFeatureSelectorNameID = 267;
            }
        );
    },
        {
        CTFeatureTypeExclusive = 1;
        CTFeatureTypeIdentifier = 17;
        CTFeatureTypeName = "Character Alternatives";
        CTFeatureTypeNameID = 262;
        CTFeatureTypeSelectors =         (
                        {
                CTFeatureSelectorDefault = 1;
                CTFeatureSelectorIdentifier = 0;
                CTFeatureSelectorName = "No Change";
                CTFeatureSelectorNameID = 264;
            },
                        {
                CTFeatureSelectorIdentifier = 1;
                CTFeatureSelectorName = "Alternate Punctuation";
                CTFeatureSelectorNameID = 263;
            },
                        {
                CTFeatureSelectorIdentifier = 2;
                CTFeatureSelectorName = "Numbers Punctuation";
                CTFeatureSelectorNameID = 265;
            }
        );
    }
)

### 查询字体计量
UIFont定义了一些访问字体计量信息的方法（当这些信息可用时）。例如ascender，capHeight，xHeight等属性都代表标准的计量信息。

## 布局文本

NSLayoutManager类对象，是Text Kit中文本展示的控制中枢，布局管理器执行以下的操作：
* 控制text storage和text container对象
* 根据字符生成文字图像
* 计算文字位置并保存信息
* 管理文字区间
* 需要时在text view上绘制文字
* 计算文本行的边界
* 控制连字符使用
* 操作文字和字符属性
在MVC范式中，layout manager充当了controller。NSTextStorage充当了Model，负责字符串及其属性，如字体、类型、颜色、大小。NSTextContainer也可以被认为是Model的一部分，因为它是文本在页面上几何布局的model。UITextView（或其他UIView对象）提供文本展示的View。NSLayoutManager作为controller提供服务，它将字符从text storage对象转换成图形，并将它们根据一个或多个text container的尺寸布局到行中，将文本在一个或多个text view对象中进行展示。

## 布局过程

布局管理器将文本布局分两个步骤：生成图形和布局图形。布局管理器“惰性”的执行这两个步骤（必要时才执行）。它生成图形和计算出布局位置后，布局管理器会缓存这些信息来改善后续调用的性能。

布局管理器缓存图形、属性和布局信息。它会监控text storage中因为修改而失效的文字图形的区间。在两种情况下字符区间会自动失效：当需要生成文字图形或者当需要布局字符图形时。你也可以手动的使字符图形或布局信息实效。当布局管理器收到需要知道一个无效区间内的文字图形或布局信息时，它会重新生成它们。

### 生成行片段边界

布局管理器将NStextContainer对象中的文字图形在行中进行布局。text container根据自身的形状和内部的非布局区域来确定每一行的位置。当一个行与非布局区域相交时，行必须被缩短或拆分。
布局管理器为给定的一行提出一个边界然后要求text container来调整到适应的边界。提出的边界可能比container更宽或更窄。布局管理器向text container发送的要求调整提出边界的方法是lineFragmentRectForProposedRect:atIndex:writingDirection:remainingRect:，这个方法根据文本布局的方向为提出的边界返回最大的可用边界。它也会返回其他剩余区域的大小，如text container上的空白或黑洞其他边上剩余的空间。

当布局管理器将文本置于边界中时会做最后一个调整。这种调整是text container进行的少量修正，称为行碎片填充，定义了每行后部的空白部分。文本根据行片段边距进行插入（边距本身不受影响）。填充允许在text container的边界一点小的调整，并使文本和其他图形的边界保持一定的距离。你可以通过修改lineFragmentPadding属性的值来修改填充。注意，行片段填充并不是边距。对于文档边距来说，你应该设置UITextView对象的位置和和大小。对于文本边距，你应该设置text view的textContainerInset属性。另外，你可以设置单独的段落缩紧使用NSMutableParagraphStyle属性如headIndent。

除了返回行片段区域外，布局管理器还会返回一个已用矩形，就是行片段边界中已经包含了要绘制的文字或其他标志的部分。按照惯例，两个矩形都包括行片段填充和行内空间（根据字体的行高计量和段落的行间距参数计算出的）。然而，段落间距（段落前和后）和文本周围添加的如可以引起文本中间被隔开的空间，只在行片段矩形中包含，但不在已使用的矩形中包括。

### 指定非布局区域（Exclusion Paths）

text container维护了一个UIBezierPath对象的数组来代表它里面的非布局区域。当布局管理器向text container发送lineFragmentRectForProposedRect:atIndex:writingDirection:remainingRect:消息，提出了一个行片段矩形和非布局区域相交时，text container返回一个调整后的不包括非布局区域的矩形。这个处理过程如下图所示：


### 指定多页和多列布局

最简单的情况中，Text Kit的每类对象都只有一个，就是一个text storage对象，一个text container，一个layout manager，如下面的图所示。当你从对象库中向IB中拖一个text view对象时，这样的设定就会被自动配置好。UITextView对象创建其他的对象并将它们连接在一起。你可以用代码管理这个过程。

下面的代码创建这些对象，这段代码可以在一个viewControllere中，假设这个viewController有一个叫textContainer的NSTextContainer属性。
NSTextStorage *textStorage = [[NSTextStorage alloc]initWithString:string];
NSLayoutManager *layoutManager = [[NSLayoutManager alloc]init];
[textStorage addLayoutManager:layoutManager];
self.textContainer = [[NSTextContainer alloc]initWithSize:self.view.bounds.size];
[layoutManager addTextContainer:self.textContainer];
UITextView *textView = [[UITextView alloc]initWithFrame:self.view.bounds textContainer:self.textContainer];
[self.view addSubview:textView];

这种只有一个text container和一个text view的结构非常受限。这种结构中，文本流在text container中定义的区域内不会被中断。遇到翻页、多列布局和其他更复杂的布局就不好使了。

通过使用多个text container，每个都联系一个text view，可以让更复杂的布局成为可能。例如，要支持翻页，应用可以配置如下图所示的文本对象：

每个text container相当于文档中的一页。你可以把展示文本的视图集成进你自定义了背景的自定义视图中。在这个自定义视图中，还可以集成一个UIScrollView对象允许用户可以在文档的页面之间进行翻页。

多列的文档也可以用相似的配置：

 这里一个页面和两个text container相联系，一个text container代表页面上的一列。每个text container控制文档的一部分。当文本展示时，文字图形首先在左上角的container中进行布局。当那个视图没有剩余的空间时，布局管理器通知他的delegate它已经填充完毕那个container。delegate可以检查一下是否还有没有布局的文本，然后根据需要添加一个text container。布局管理器继续在下一个container中执行文本布局，完成后继续提醒delegate，以此类推。同样的，这里用一个自定义的视图（蓝色矩形边框）来为这些文本列提供画布。

注意，这里不仅可以有多个text container，也可以用多个NSLayoutManager对象来访问相同的text storage。下图展示了多个布局管理器的情况。这种配置可以为相同的文本的展示提供多个视图。如果用户修改了上面的视图的文本，同样的修改会立即反应在下面的视图中（假设修改的位置在下面的视图的可视区域内）。
