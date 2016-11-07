---
layout: post
title: "使用Text Kit来绘制和管理文本"
date: 2016-11-04 08:29:59 +0800
categories:
---

引言：本文来自Apple官方文档《Text Programming Guide for iOS: Using Text Kit to Draw and Manager》，可能会由于本人英语水平（未过英语四级考试）、理解能力和文档时效等问题造成与原文出入较大，阅读时请慎重！如遇到问题，请在评论中指出。

UIKit 框架中提供了一些用来在App的用户界面中展示文本的类：UITextView, UITextField, UILabel 和 UIWebView。由 UITextView 类创建的文本视图意在用来展示大量的文本。UITextView 底层有一个非常强大的布局引擎叫做 Text Kit。如果你需要自定义或者介入这个布局过程，你可以使用 Text Kit。对于更细粒度的文本自定义解决方案的需求，可以参考《Lower Level Text-handling Technologies》中关于更底层技术的描述。

TextKit是UIKit框架提供的高品质排版服务的类和协议的集合。TextKit构建于CoreText之上，它同样提供了速度与强大。
TextKit框架在iOS 图形文本框架中的位置：
![](../images/utktdamt/9-1.png =500x100)

## TextKit主要的成员对象

TextKit主要对象之间的数据流图如下。Text views是UITextView类的实例，Text containers是NSTextContainer类的实例，layout manager是NSLayoutManager类的实例，Text storage是NSTextStorage类实例。在TextKit中，一个NSTextStorage对象存储文本以供UITextView对象显示和NSLayoutManager对象在一个NSTextContainer对象上布局。

NSTextContainer对象定义了一个可供文本布局的区域。text container默认定义一个方形区域，但可以通过创建NSTextContainer的子类来定义其他形状如圆、五边形或不规则的形状。text container不仅定义了文本区域的边界，同时它还维护了一个Bezier paths数组定义了区域中不能布局文本的地方（例外区）。当布局发生时，文本在例外区以外布局，这样就为包含图像和其他非文本的元素提供了一种方式。

NSTextStorage为Text Kit扩展的文本处理系统定义了存储机制基础。NSTextStorage是NSMutableAttributedString的一个子类，可用来储存文本系统处理的文本和属性。它保证了文本和属性在编辑操作中可以保持协调一致。除了保存文本，NSTextStorage对象还管理了NSLayoutManager对象的集合，当文本和属性发生改变时通知NSLayoutManager来做出改变。

NSLayoutManager对象筹划其他文本处理对象的操作。它在将NSTextStorage中的数据转换为视图的显示区域的操作中进行调解。它将Unicode码映射为NSTextContainer布局区域上的文字。

注意： NSLayoutManager、NSTextStorage和NSTextContainer可以在子线程中使用，但要保证只在单一的线程中被使用。

## 文本属性

Text Kit处理三种文本属性：文字属性，段落属性和文档属性。文字属性包括字体、颜色和下标等可以在单独的或一段文字上设定的特性。段落属性如缩进、tabs，和行间距。文档属性包括纸张大小，边距和视图缩放百分比等特性。

### 字符属性

attributed string通过字典中的键值对的方式存储文字属性。键为属性的名称，用一个字符串常量（如NSFontAttributeName）来表示。下图展示了一个属性字典应用与一个属性字符串中的情况。

理论上，属性化字符串中每个字符都有一个属性字典与之对应。然而通常一个属性字典应用于一串字符上。NSAttributedString类提供了根据字符的序号获取对应的属性字典和该属性适用的区间的方法如：attributesAtIndex:effectiveRange:.

你可以向NSTextStorage中合适的字符区间通过NSMutableAttributedString的addAttribute:value:range:添加属性，你也可以创建一个包含自定义的属性的键值对的字典然后通过addAttributes:range:方法添加到一个区间的字符上。要使用自定义的属性时，你需要实现一个NSLayoutManager的子类。你自定义的子类应该重写drawGlyphsForGlyphRange:atPoint:方法。在这个方法里可以先调用父类相应的方法来绘制文字然后在上面加上自定义的属性，或者完全使用自己的方式绘制。

### 段落属性

段落属性作用于layout manager安排页面上段落中的行。文本系统中的段落属性抽象为NSParagraphStyle类。预定义的文字属性中的NSParagraphStyleAttributeName键指向一个NSParagraphStyle对象，包含了对应区间的文字的段落属性。属性修复机制保证了每个段落中的文本只有一个NSParagraphStyle对象与之对应。
段落属性包含如alignment，tab stops，line-breaking mode，和line spacing（leading）等特性。

### 文档属性

文档属性属于文档整体。文档属性包含纸张大小，边距和视图缩放比例等特性。尽管文本系统中没有内置的存储文档属性的机制，NSAttributedString的初始化方法如initWithRTF:documentAttributes:可以让你传递一个属性字典来控制RTF或HTML数据的文档属性。相反的，写入RTF数据的方法如RTFFromRange:documentAttributes:也让你传一个属性字典。

### 属性修正

编辑属性字符串时会引发需要被属性修复清理掉的属性矛盾。UIKit扩展了NSMutableAttributedString，定义了fixAttributesInRange:方法来修复附件、字符和段落属性中的矛盾。这保证了当一个附件所绑定的字符被删除时这个附件也被移除、字符属性只应用在支持（如字体）的字符上，和段落属性在整个段落中贯穿始终。

## 通过编程改变文本存储

NSTextStorage对象在Text Kit中用来代表字符数据。这些数据的格式是一个用unicode编码的连接了属性的字符序列。表示属性化字符串的类有NSAttributedString和NSMutableAttributedString，和子类NSTextStorage。如字符属性中所讲，文本中的每一段文字都有一个键值对字典与其联系。键为一个属性的名称，对应的值就是对应的属性值。

有三个步骤编程的方式编辑text storage对象。第一步是向其发送一个beginEditing消息来宣布一组修改。
在第二个步骤中，你向它发送一些编辑消息，例如replaceCharactersInRange:withString:和setAttributes:range:,来反映字符或属性的改变。每次发送这些消息时，text storage对象调用edited:range:changeInLength:来跟踪从收到beginEditing消息后受影响的字符区间。
在第三步中，当你完成了对text storage对象的修改，向其发送endEditing消息。这会使其调用其delegate的textStorage:willProcessEditing:range:changeInLength:和其自身的processEditing方法，修复改变区间的属性冲突。

修正完属性后，text storage对象向其delegate发送textStorage:didProcessEditing:range:changeInLength:方法，给delegate一个合适时机来验证属性的改变。（尽管delegate在这个方法中可以修改text storage对象的字符属性，但这会使text storage处于一个前后矛盾的状态。）最后，text storage对象向每个联系的layout manager发送processEditingForTextStorage:edited:range:changeInLength:invalidateRange:消息，layout manager们轮流的使用这些信息来重新计算他们的文字位置并在必要时重新展示。

## 使用字体对象

计算机自体是如OpenType或TrueType的格式的数据文件，包含了图形信息的集合和图形渲染需要的其他信息。UIFont提供了获取和设置字体信息的接口。一个UIFont实例提供了访问字符和图形的入口。Text Kit结合字符信息和字体信息来选择布局要使用的图形。你可以将字体对象传递给接收字体作为参数的方法中来使用。字体对象是不可变的，所以在应用的多个线程中使用是安全的。

你不能使用alloc和init方法来创建UIFont对象；而要用preferredFontForTextStyle:或fontWithName:size:来创建。也可以用font descriptor通过fontWithDescriptor:size:来创建字体。这些方法会先从已存在的字体对象中查找是否存在指定特征的字体对象，如果有就返回，否则，创建一个合适的字体对象。

### 文本类型

Text styles，在iOS 7带来的由被称为动态类型的机制实现的对想要使用的字体的语义描述。Text styles于UIFontDescriptor.h中用常量进行组织和定义。text style描述的font可能考虑一些动态的因素如用户的内容大小便好（通过UIApplication的preferredContentSizeCategory属性表示）与实际使用的不同。可以通过向UIFont的preferredFontForTextStyle:方法传递常量来获得字体对象。通过向UIFontDescriptor的preferredFontDescriptorWithTextStyle:方法传递相应的常量来获得font descriptor对象。
常量
UIFontTextStyleHeadline           用于headings的字体
UIFontTextStyleSubheadline     用于subheads的字体
UIFontTextStyleBody                用于body文本的字体
UIFontTextStyleFootnote          用于footnotes的字体
UIFontTextStyleCaption1          用于标准的首部的字体
UIFontTextStyleCaption2          用于副首部的字体

Text styles通过动态类型机制为应用带来了许多改善你的文本的阅读性的特性，动态类型可以协调并响应用户的偏好。当你调用preferredFontForTextStyle:时，包含了用户偏好和当前环境和使用指定的text style等特征的字体被返回。

根据text style得到的字体大小应该被用于应用中所有的文本中，除了界面元素如按钮、标签等。当然你需要合适你应用的text style。同时，观察UIContentSizeCategoryDidChangeNotification也很重要，这样可以在用户修改了内容尺寸类别时重新布局。当你的应用收到这个通知时，它应该向Auto Layout下的view发送invalidateIntrinsicContentSize或者向手动布局的用户界面元素发送setNeedsLayout消息。同时要使先前的font或font descriptor失效，然后获取新的。

### 使用字体描述

字体描述，UIFontDescriptor类的实例，提供了通过一个属性字典描述字体的方法，并且可以用它来创建UIFont对象。你可以从一个字体描述创建一个UIFont对象，也可以从UIFont对象得到一个描述，还可以改变这个描述然后用它生成一个新的字体对象。你也可以用一个字体描述来指定应用提供的自定义的字体。

字体描述可以被存档，这样可以高效的使用text style。你不应该使用text style来缓存字体对象因为text style是动态的（根据用户的偏好不同会改变）。但你可以缓存字体描述，然后稍后用它再创建相同特征的字体对象。

你可以使用字体描述来查询系统中符合特定属性的可用字体，然后创建符合这些属性（名称、特点、语言或其他特性）的字体对象。例如，你可用使用一个字体描述来获取所有的符合给定的字体簇名称的字体，使用CSS标准定义的簇名称：
UIFontDescriptor *helveticaNeueFamily = [UIFontDescriptor fontDescriptorWithFontAttributes:@{UIFontDescriptorFamilyAttribute: @“Helvetica Neue”}];
NSArray *matches = [helvetecaNeueFamily matchingFontDescriptorsWithMandatoryKeys:nil];
matchingFontDescriptorsWithMandatoryKeys:方法返回系统中所有的Helvetica Neue字体的描述，如HelveticaNeue, HelveticaNeue-Medium, HelveticaNeue-Light, HelveticaNeue-Thin,等。

你可以修改preferredFontForTextStyle:方法返回的字体，通过应用形状特征，如粗体、斜体、宽松、紧凑：
UIFontDescriptor *fontDescriptor = [UIFontDescriptor preferredFontDescriptorWithTextStyle:UIFontTextStyleBody];
UIFontDescriptor *boldFontDescriptor = [fontDescriptor fontDescriptorWithSymbolicTraits:UIFontDescriptorTraitBold];
UIFont *boldFont = [UIFont fontWithDescriptor:boldFontDescriptor size:0.0];
这段代码中最后一句的size参数传0.0是指定使用字体描述中的字体大小，在动态类型机制中，这是被提倡的。

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
