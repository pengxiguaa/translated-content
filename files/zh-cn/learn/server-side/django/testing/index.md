---
title: 'Django 教程 10: 测试 Django 网页应用'
slug: learn/Server-side/Django/Testing
translation_of: Learn/Server-side/Django/Testing
---
<div>{{LearnSidebar}}</div>

<div>{{PreviousMenuNext("Learn/Server-side/Django/Forms", "Learn/Server-side/Django/Deployment", "Learn/Server-side/Django")}}</div>

<p>随着网站的增长，他们越来越难以手动测试。不仅要进行更多的测试，而且随着组件之间的交互变得越来越复杂，一个区域的小改变可能会影响到其他区域，所以需要做更多的改变来确保一切正常运行，并且在进行更多更改时不会引入错误。减轻这些问题的一种方法是编写自动化测试，每当您进行更改时，都可以轻松可靠地运行测试。本教程演示如何使用 Django 的测试框架自动化您的网站的<em>单元测试</em>。</p>

<table class="learn-box standard-table">
 <tbody>
  <tr>
   <th scope="row">先决条件：</th>
   <td>完成之前的所有教程主题，包括 Django 教程 9：使用表单。</td>
  </tr>
  <tr>
   <th scope="row">目标：</th>
   <td>了解如何为基于 Django 的网站编写单元测试。</td>
  </tr>
 </tbody>
</table>

<h2 id="概览">概览</h2>

<p>LocalLibrary 目前有页面显示所有书本和作者的列表，书本和作者项目的详细视图，续借<code>BookInstances</code>的页面，以及创建，更新和删除作者项目的页面（如果您完成了<strong>Django 教程 9：使用表单</strong>中的自我挑战，也可以创建，更新和删除书本记录）。即使使用这个相对较小的站点，手动导航到每个页面，并且表面地检查一切是否按预期工作，可能需要几分钟。当我们进行更改，并扩展网站时，手动检查所有内容“正常”工作所需的时间只会增长。如果我们继续这样做，最终我们将花费大部分时间进行测试，并且很少有时间来改进我们的代码。</p>

<p>自动化测试可以真正帮助解决这个问题！显而易见的好处，是它们可以比手动测试运行得更快，可以测试更底层级别的细节，并且每次都测试完全相同的功能（人类测试员远远没有这么可靠！）因为它们很快速，自动化的测试可以更频繁地执行，如果测试失败，他们会指出代码未按预期执行的位置。</p>

<p>此外，自动化测试可以充当代码的第一个真实“用户”，迫使您严格定义和记录网站的行为方式。它们通常是您的代码示例，和文档的基础。由于这些原因，一些软件开发过程，从测试定义和实现开始，之后编写代码以匹配所需的行为（例如，测试驱动<a href="https://en.wikipedia.org/wiki/Test-driven_development">test-driven</a> 和行为驱动 <a href="https://en.wikipedia.org/wiki/Behavior-driven_development">behaviour-driven</a>的开发）。</p>

<p>本教程通过向 LocalLibrary 网站添加大量测试，来演示如何为 Django 编写自动化测试。</p>

<h3 id="测试的类型">测试的类型</h3>

<p>测试和测试方法有许多类型，级别和分类。最重要的自动化测试是：</p>

<dl>
 <dt>单元测试 Unit tests</dt>
 <dd>验证各个组件的功能行为，通常是类别和功能级别。</dd>
 <dt>回归测试</dt>
 <dd>测试重现历史错误。最初运行每个测试，以验证错误是否已修复，然后重新运行，以确保在以后更改代码之后，未重新引入该错误。</dd>
 <dt>集成测试</dt>
 <dd>验证组件分组在一起使用时的工作方式。集成测试了解组件之间所需的交互，但不一定了解每个组件的内部操作。它们可能涵盖整个网站的简单组件分组。</dd>
</dl>

<div class="note">
<p><strong>备注：</strong> 其他常见类型的测试，包括黑盒，白盒，手动，自动，金丝雀，烟雾，一致性，验收，功能，系统，性能，负载和压力测试。查找它们以获取更多信息。</p>
</div>

<h3 id="Django为测试提供了什么？">Django 为测试提供了什么？</h3>

<p>测试网站是一项复杂的任务，因为它由多层逻辑组成 - 从 HTTP 级请求处理，查询模型，到表单验证和处理，以及模板呈现。</p>

<p>Django 提供了一个测试框架，其中包含基于 Python 标准<code><a href="https://docs.python.org/3/library/unittest.html#module-unittest">unittest</a></code>库的小型层次结构。尽管名称如此，但该测试框架适用于单元测试和集成测试。Django 框架添加了 API 方法和工具，以帮助测试 Web 和 Django 特定的行为。这允许您模拟请求，插入测试数据以及检查应用程序的输出。Django 还提供了一个 API（<a href="https://docs.djangoproject.com/en/2.0/topics/testing/tools/#liveservertestcase">LiveServerTestCase</a>）和<a href="https://docs.djangoproject.com/en/2.0/topics/testing/advanced/#other-testing-frameworks">使用不同测试框架</a>的工具，例如，您可以与流行的 <a href="/en-US/docs/Learn/Tools_and_testing/Cross_browser_testing/Your_own_automation_environment">Selenium</a> 框架集成，以模拟用户与实时浏览器交互。</p>

<p>要编写测试，您可以从任何 Django（或 unittest）测试基类（<a href="https://docs.djangoproject.com/en/2.0/topics/testing/tools/#simpletestcase">SimpleTestCase</a>, <a href="https://docs.djangoproject.com/en/2.0/topics/testing/tools/#transactiontestcase">TransactionTestCase</a>, <a href="https://docs.djangoproject.com/en/2.0/topics/testing/tools/#testcase">TestCase</a>, <a href="https://docs.djangoproject.com/en/2.0/topics/testing/tools/#liveservertestcase">LiveServerTestCase</a>）派生，然后编写单独的方法，来检查特定功能，是否按预期工作（测试使用“assert”方法来测试表达式导致 <code>True</code>或 <code>False</code>值，或者两个值相等，等等。）当您开始测试运行时，框架将在派生类中执行所选的测试方法。测试方法独立运行，具有在类中定义的常见设置和/或拆卸行为，如下所示。</p>

<pre class="brush: python">class YourTestClass(TestCase):

    def setUp(self):
        #Setup run before every test method.
        pass

    def tearDown(self):
        #Clean up run after every test method.
        pass

    def test_something_that_will_pass(self):
        self.assertFalse(False)

    def test_something_that_will_fail(self):
        self.assertTrue(False)
</pre>

<p>大多数测试的最佳基类是 <a href="https://docs.djangoproject.com/en/2.0/topics/testing/tools/#testcase">django.test.TestCase</a>。此测试类在运行测试之前，创建一个干净的数据库，并在自己的事务中，运行每个测试函数。该类还拥有一个<a href="https://docs.djangoproject.com/en/2.0/topics/testing/tools/#django.test.Client">测试客户端</a>，您可以使用该客户端，模拟在视图级别与代码交互的用户。在下面的部分中，我们将集中讨论使用此<a href="https://docs.djangoproject.com/en/2.0/topics/testing/tools/#testcase">TestCase</a> 基类创建的单元测试。</p>

<div class="note">
<p><strong>备注：</strong> <a href="https://docs.djangoproject.com/en/2.0/topics/testing/tools/#testcase">django.test.TestCase</a> 类非常方便，但可能会导致某些测试，比它们需要的速度慢（并非每个测试，都需要设置自己的数据库，或模拟视图交互）。一旦熟悉了这个类可以做什么，您可能希望用可以用更简单的测试类，替换一些测试。</p>
</div>

<h3 id="你应该测试什么？">你应该测试什么？</h3>

<p>您应该测试自己代码的所有方面，但不要测试 Python 或 Django 的一部分提供的任何库或功能。</p>

<p>例如，考虑下面定义的 <code>Author</code>模型。您不需要显式测试 <code>first_name</code> 和 <code>last_name</code> 是否已在数据库中正确储存为<code>CharField</code>，因为这是 Django 定义的内容（当然，在实践中，您将不可避免地在开发期间测试此功能）。你也不需要测试<code>date_of_birth</code>是否已被验证为日期字段，因为这也是 Django 中实现的东西。</p>

<p>但是，您应该检查用于标签的文本（名字，姓氏，出生日期，死亡），以及为文本分配的字段大小（100 个字符），因为这些是您的设计的一部分，可能会在将来被打破/改变。</p>

<pre class="brush: python">class Author(models.Model):
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    date_of_birth = models.DateField(null=True, blank=True)
    date_of_death = models.DateField('Died', null=True, blank=True)

    def get_absolute_url(self):
        return reverse('author-detail', args=[str(self.id)])

    def __str__(self):
        return '%s, %s' % (self.last_name, self.first_name)</pre>

<p>同样，您应该检查自定义方法 <code>get_absolute_url()</code> 和 <code>__str__()</code> 是否符合要求，因为它们是您的代码/业务逻辑。在<code>get_absolute_url()</code>的情况下，您可以相信 Django <code>reverse()</code>方法已经正确实现，因此您正在测试的是实际上已经定义了关联的视图。</p>

<p> </p>

<div class="note">
<p><strong>备注：</strong> 精明的读者可能会注意到，我们也希望将出生和死亡的日期限制在合理的值，并检查出生后是否死亡。在 Django 中，此约束将添加到表单类中（尽管您可以为字段定义验证器，这些字段似乎仅在表单级别使用，而不是在模型级别使用）。</p>
</div>

<p>考虑到这些，让我们开始研究如何定义和运行测试。</p>

<h2 id="测试结构概述">测试结构概述</h2>

<p>在我们详细讨论“测试内容”之前，让我们先简要介绍一下测试的定位和方式。</p>

<p>Django 使用 unittest 模块的<a href="https://docs.python.org/3/library/unittest.html#unittest-test-discovery">内置测试查找</a>，它将在任何使用模式<strong>test*.py </strong>命名的文件中，查找当前工作目录下的测试。如果您正确命名文件，则可以使用您喜欢的任何结构。我们建议您为测试代码创建一个模块，并为模型，视图，表单和您需要测试的任何其他类型的代码，分别创建文件。例如：</p>

<pre>catalog/
  /tests/
    __init__.py
    test_models.py
    test_forms.py
    test_views.py
</pre>

<p>在 LocalLibrary 项目中，创建如上所示的文件结构。<strong>__init__.py </strong>应该是一个空文件（这告诉 Python 该目录是一个套件包）。您可以通过复制和重命名框架测试文件<strong>/catalog/tests.py</strong>，来创建三个测试文件。</p>

<p> </p>

<div class="note">
<p><strong>备注：</strong> 我们构建 Django 骨架网站时，会自动创建骨架测试文件<strong>/catalog/tests.py</strong> 。将所有测试放入其中是完全“合法的”，但如果测试正确，您将很快得到一个非常庞大且难以管理的测试文件。</p>

<p>删除骨架文件，因为我们不需要它。</p>
</div>

<p>打开 <strong>/catalog/tests/test_models.py</strong>。该文件应导入<code>django.test.TestCase</code>，如下所示：</p>

<pre class="brush: python">from django.test import TestCase

# Create your tests here.
</pre>

<p>通常，您将为要测试的每个模型/视图/表单添加测试类别，并使用个别方法来测试特定功能。在其他情况下，您可能希望有一个分开的类别，来测试特定用例，使用个别的测试函数，来测试该用例的各个方面（例如，测试模型字段已正确验证的类，以及测试每个可能的失败案例的函数）。相同地，这样的结构非常适合您，但最好您能保持一致。</p>

<p>将下面的测试类别，添加到文件的底部。该类别演示了，如何通过派生<code>TestCase</code>，构建测试用例类。</p>

<pre class="brush: python">class YourTestClass(TestCase):

    @classmethod
    def setUpTestData(cls):
        print("setUpTestData: Run once to set up non-modified data for all class methods.")
        pass

    def setUp(self):
        print("setUp: Run once for every test method to setup clean data.")
        pass

    def test_false_is_false(self):
        print("Method: test_false_is_false.")
        self.assertFalse(False)

    def test_false_is_true(self):
        print("Method: test_false_is_true.")
        self.assertTrue(False)

    def test_one_plus_one_equals_two(self):
        print("Method: test_one_plus_one_equals_two.")
        self.assertEqual(1 + 1, 2)</pre>

<p>新的类别定义了两个可用于测试之前的配置的方法（例如，创建测试所需的任何模型或其他对象）：</p>

<ul>
 <li><code>setUpTestData()</code> 用于类级别设置，在测试运行开始的时侯，会调用一次。您可以使用它来创建在任何测试方法中，都不会修改或更改的对象。</li>
 <li><code>setUp()</code> 在每个测试函数之前被调用，以设置可能被测试修改的任何对象（每个测试函数，都将获得这些对象的“新”版本）。</li>
</ul>

<div class="note">
<p><strong>备注：</strong> 测试类别还有一个我们还没有使用的<code>tearDown()</code>方法。此方法对数据库测试不是特别有用，因为<code>TestCase</code>基类会为您处理数据库拆卸。</p>
</div>

<p>下面我们有一些测试方法，它们使用 <code>Assert </code>函数来测试条件是真，假或相等（<code>AssertTrue</code>, <code>AssertFalse</code>, <code>AssertEqual</code>）。如果条件评估不如预期，则测试将失败，并将错误报告给控制台。</p>

<p><code>AssertTrue</code>, <code>AssertFalse</code>, <code>AssertEqual </code>是 <strong>unittest </strong>提供的标准断言。框架中还有其他标准断言，还有 <a href="https://docs.djangoproject.com/en/2.0/topics/testing/tools/#assertions">Django 特定的断言</a>，来测试视图是否重定向（<code>assertRedirects</code>），或测试是否已使用特定模板（<code>assertTemplateUsed</code>）等。</p>

<div class="note">
<p><strong>备注：</strong> 您通常<strong>不应</strong>在测试中包含<strong>print()</strong> 函数，如上所示。我们这样做，只是为了让您可以看到在控制台中，调用设置功能的顺序（在下一节中）。</p>
</div>

<h2 id="如何运行测试">如何运行测试</h2>

<p>要运行所有测试，最简单的方法，是使用以下命令：</p>

<pre class="brush: bash">python3 manage.py test</pre>

<p>这将查找当前目录下，使用模式 <strong>test*.py</strong> 命名的所有文件，并运行使用适当基类定义的所有测试（这里我们有许多测试文件，但只有 <strong>/catalog/tests/test_models.py</strong> 目前包含任何测试。）。默认情况下，测试将仅单独报告测试失败，然后是测试摘要。</p>

<p> </p>

<div class="note">
<p><strong>备注：</strong> 如果您收到类似于以下内容的错误：<code>ValueError: Missing staticfiles manifest entry ...</code> 这可能是因为默认情况下，测试不会运行 collectstatic，而您的应用程序正在使用需要它的储存类别（有关更多信息，请参阅 <a href="https://docs.djangoproject.com/en/2.0/ref/contrib/staticfiles/#django.contrib.staticfiles.storage.ManifestStaticFilesStorage.manifest_strict">manifest_strict</a>）。有许多方法可以解决这个问题 - 最简单的方法，是在运行测试之前，简单地运行 collectstatic：</p>

<pre class="brush: bash">python3 manage.py collectstatic
</pre>
</div>

<p>在 LocalLibrary 的根目录中，运行测试。您应该看到如下所示的输出。</p>

<pre class="brush: bash">&gt;python3 manage.py test

Creating test database for alias 'default'...
<strong>setUpTestData: Run once to set up non-modified data for all class methods.
setUp: Run once for every test method to setup clean data.
Method: test_false_is_false.
.setUp: Run once for every test method to setup clean data.
Method: test_false_is_true.
FsetUp: Run once for every test method to setup clean data.
Method: test_one_plus_one_equals_two.</strong>
.
======================================================================
FAIL: test_false_is_true (catalog.tests.tests_models.YourTestClass)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "D:\Github\django_tmp\library_w_t_2\locallibrary\catalog\tests\tests_models.py", line 22, in test_false_is_true
    self.assertTrue(False)
AssertionError: False is not true

----------------------------------------------------------------------
Ran 3 tests in 0.075s

FAILED (failures=1)
Destroying test database for alias 'default'...</pre>

<p>在这里，我们看到有一个测试失败，我们可以确切地看到哪个函数失败了、为什么失败（这个失败是预期的，因为 <code>False</code>不是 <code>True</code>！）。</p>

<div class="note">
<p><strong>备注：</strong> 从上面的测试输出中，学到的最重要事情是，如果为对象和方法使用描述性/信息性名称，它会更有价值。</p>
</div>

<p>上面以<strong>粗体</strong>显示的文本，通常不会出现在测试输出中（这是由我们的测试中的<code>print()</code>函数生成的）。这显示了如何为类调用<code>setUpTestData()</code>方法，并在每个方法之前调用<code>setUp()</code>。</p>

<p>接下来的部分，将介绍如何运行特定测试，以及如何控制测试显示的信息量。</p>

<h3 id="显示更多测试信息">显示更多测试信息</h3>

<p>如果您想获得有关测试运行的更多信息，可以更改详细程度。例如，要列出测试成功和失败（以及有关如何设置测试数据库的大量信息），您可以将详细程度设置为“2”，如下所示：</p>

<pre class="brush: bash">python3 manage.py test --verbosity 2</pre>

<p>允许的详细级别为 0, 1 ,2 和 3，默认值为“1”。</p>

<h3 id="运行特定测试">运行特定测试</h3>

<p>如果要运行测试的子集，可以通过指定包，模块，<code>TestCase</code>子类或方法的完整路径​​（包含点）来执行此操作：</p>

<pre class="brush: bash">python3 manage.py test catalog.tests   # Run the specified module
python3 manage.py test catalog.tests.test_models  # Run the specified module
python3 manage.py test catalog.tests.test_models.YourTestClass # Run the specified class
python3 manage.py test catalog.tests.test_models.YourTestClass.test_one_plus_one_equals_two  # Run the specified method
</pre>

<h2 id="LocalLibrary_测试">LocalLibrary 测试</h2>

<p>现在我们知道，如何运行我们的测试，以及我们需要测试哪些东西，让我们看一些实际的例子。</p>

<div class="note">
<p><strong>备注：</strong> 我们不会编写所有可能的测试，但这应该可以让您了解测试的工作原理，以及您可以做些什么。</p>
</div>

<h3 id="模型">模型</h3>

<p>如上所述，我们应该测试我们设计的任何内容，或由我们编写的代码定义的内容，而不是已经由 Django 或 Python 开发团队测试过的库/代码。</p>

<p>例如，请考虑下面的作者模型<code> Author</code>。在这里，我们应该测试所有字段的标签，因为即使我们没有明确指定它们中的大部分，我们也有一个设计，说明这些值应该是什么。如果我们不测试值，那么我们不知道字段标签，是否具有其预期值。同样，虽然我们相信 Django 会创建一个指定长度的字段，但值得为这个长度指定一个测试，以确保它按计划实现。</p>

<pre class="brush: python">class Author(models.Model):
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    date_of_birth = models.DateField(null=True, blank=True)
    date_of_death = models.DateField('Died', null=True, blank=True)

    def get_absolute_url(self):
        return reverse('author-detail', args=[str(self.id)])

    def __str__(self):
        return '%s, %s' % (self.last_name, self.first_name)</pre>

<p>打开我们的 <strong>/catalog/tests/test_models.py</strong>，并用 <code>Author</code>模型的以下测试代码，替换任何现有代码。</p>

<p>在这里，您将看到我们首先导入 <code>TestCase</code>，并使用描述性名称，从中派生我们的测试类（<code>AuthorModelTest</code>），以便我们可以轻松识别测试输出中的任何失败测试。然后我们调用<code>setUpTestData()</code>，来创建一个我们将使用，但不在任何测试中修改的作者对象。</p>

<pre class="brush: python">from django.test import TestCase

# Create your tests here.

from catalog.models import Author

class AuthorModelTest(TestCase):

    @classmethod
    def setUpTestData(cls):
        #Set up non-modified objects used by all test methods
        Author.objects.create(first_name='Big', last_name='Bob')

    def test_first_name_label(self):
        author=Author.objects.get(id=1)
        field_label = author._meta.get_field('first_name').verbose_name
        self.assertEquals(field_label,'first name')

    def test_date_of_death_label(self):
        author=Author.objects.get(id=1)
        field_label = author._meta.get_field('date_of_death').verbose_name
        self.assertEquals(field_label,'died')

    def test_first_name_max_length(self):
        author=Author.objects.get(id=1)
        max_length = author._meta.get_field('first_name').max_length
        self.assertEquals(max_length,100)

    def test_object_name_is_last_name_comma_first_name(self):
        author=Author.objects.get(id=1)
        expected_object_name = '%s, %s' % (author.last_name, author.first_name)
        self.assertEquals(expected_object_name,str(author))

    def test_get_absolute_url(self):
        author=Author.objects.get(id=1)
        #This will also fail if the urlconf is not defined.
        self.assertEquals(author.get_absolute_url(),'/catalog/author/1')</pre>

<p>字段测试检查字段标签（<code>verbose_name</code>）的值，以及字符字段的大小，是否符合预期。这些方法都有描述性名称，并遵循相同的模式：</p>

<pre class="brush: python">author=Author.objects.get(id=1)   # Get an author object to test
field_label = author._meta.get_field('first_name').verbose_name   # Get the metadata for the required field and use it to query the required field data
self.assertEquals(field_label,'first name')  # Compare the value to the expected result</pre>

<p>有趣的事情是：</p>

<ul>
 <li>我们无法使用 <code>author.first_name.verbose_name</code>直接获取 <code>verbose_name</code>，因为<code>author.first_name </code>是一个字符串（不是我们可以用来访问其属性的<code>first_name</code> 对象的句柄）。取而代之的是，我们需要使用作者的 <code>_meta</code>属性，来获取字段的实例，并使用它来查询其他信息。<br>
     </li>
 <li>我们选择使用 <code>assertEquals(field_label,'first name')</code> ，而不是<code>assertTrue(field_label == 'first name')</code>。这样做的原因是，如果测试失败，前者的输出，会告诉您标签实际上是什么，这使得调试问题变得更容易一些。</li>
</ul>

<div class="note">
<p><strong>备注：</strong> 已省略对<code>last_name</code> 和 <code>date_of_birth</code>标签的测试，以及 <code>last_name</code>字段长度的测试。现在按照上面显示的命名约定和方法，添加您自己的版本。</p>
</div>

<p>我们还需要测试我们的自定义方法。这些基本上只是检查对象名称，是否按照我们的预期，使用“姓氏”，“名字”格式构建，并且我们为<code>Author</code>获取的 URL，是我们所期望的。</p>

<pre class="brush: python">def test_object_name_is_last_name_comma_first_name(self):
    author=Author.objects.get(id=1)
    expected_object_name = '%s, %s' % (author.last_name, author.first_name)
    self.assertEquals(expected_object_name,str(author))

def test_get_absolute_url(self):
    author=Author.objects.get(id=1)
    #This will also fail if the urlconf is not defined.
    self.assertEquals(author.get_absolute_url(),'/catalog/author/1')</pre>

<p>立即运行测试。如果您按照模型教程中的描述，创建了作者模型，则很可能会出现<code>date_of_death</code>标签的错误，如下所示。测试失败，是因为它写的是期望标签定义遵循 Django 的约定，即没有大写标签的第一个字母（Django 会为你做这个）。</p>

<pre class="brush: bash">======================================================================
FAIL: test_date_of_death_label (catalog.tests.test_models.AuthorModelTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "D:\...\locallibrary\catalog\tests\test_models.py", line 32, in test_date_of_death_label
    self.assertEquals(field_label,'died')
AssertionError: 'Died' != 'died'
- Died
? ^
+ died
? ^</pre>

<p>这是一个非常小的错误，但它确实强调了，编写测试如何能够更彻底地检查，您可能做出的任何假设。</p>

<div class="note">
<p><strong>备注：</strong> 将 date_of_death 字段（/catalog/models.py）的标签更改为“death”并重新运行测试。</p>
</div>

<p>用于测试其他模型的模式，也类似于此，因此我们不会继续进一步讨论这些模式。请随意为其他模型，创建您自己的测试。</p>

<h3 id="表单">表单</h3>

<p>测试表单的理念，与测试模型的理念相同；您需要测试您编码、或设计指定的任何内容，但不测试底层框架，和其他第三方库的行为。</p>

<p>通常，这意味着您应该测试表单，是否包含您想要的字段，并使用适当的标签和帮助文本，显示这些字段。您无需验证 Django 是否正确验证了字段类型（除非您创建了自己的自定义字段和验证） - 即您不需要测试电子邮件字段，是否只接受电子邮件。但是，您需要测试，您希望在字段上执行的任何其他验证，以及您的代码将为错误生成的任何消息。</p>

<p>考虑我们更新书本的表格。这只有一个继续借阅的日期字段，它将包含我们需要验证的标签，和帮助文本。</p>

<pre class="brush: python">class RenewBookForm(forms.Form):
    """
    Form for a librarian to renew books.
    """
    renewal_date = forms.DateField(help_text="Enter a date between now and 4 weeks (default 3).")

    def clean_renewal_date(self):
        data = self.cleaned_data['renewal_date']

        #Check date is not in past.
        if data &lt; datetime.date.today():
            raise ValidationError(_('Invalid date - renewal in past'))
        #Check date is in range librarian allowed to change (+4 weeks)
        if data &gt; datetime.date.today() + datetime.timedelta(weeks=4):
            raise ValidationError(_('Invalid date - renewal more than 4 weeks ahead'))

        # Remember to always return the cleaned data.
        return data</pre>

<p>打开我们的<strong> /catalog/tests/test_forms.py</strong> 文件，并用<code>RenewBookForm</code>表单的以下测试代码，替换任何现有代码。我们首先导入我们的表单，和一些 Python 和 Django 库，以帮助测试与时间相关的功能。然后，我们以与模型相同的方式，声明我们的表单测试类，使用我们的 <code>TestCase</code> 派生测试类的描述性名称。</p>

<pre class="brush: python">from django.test import TestCase

# Create your tests here.

import datetime
from django.utils import timezone
from catalog.forms import RenewBookForm

class RenewBookFormTest(TestCase):

    def test_renew_form_date_field_label(self):
        form = RenewBookForm()
        self.assertTrue(form.fields['renewal_date'].label == None or form.fields['renewal_date'].label == 'renewal date')

    def test_renew_form_date_field_help_text(self):
        form = RenewBookForm()
        self.assertEqual(form.fields['renewal_date'].help_text,'Enter a date between now and 4 weeks (default 3).')

    def test_renew_form_date_in_past(self):
        date = datetime.date.today() - datetime.timedelta(days=1)
        form_data = {'renewal_date': date}
        form = RenewBookForm(data=form_data)
        self.assertFalse(form.is_valid())

    def test_renew_form_date_too_far_in_future(self):
        date = datetime.date.today() + datetime.timedelta(weeks=4) + datetime.timedelta(days=1)
        form_data = {'renewal_date': date}
        form = RenewBookForm(data=form_data)
        self.assertFalse(form.is_valid())

    def test_renew_form_date_today(self):
        date = datetime.date.today()
        form_data = {'renewal_date': date}
        form = RenewBookForm(data=form_data)
        self.assertTrue(form.is_valid())

    def test_renew_form_date_max(self):
        date = timezone.now() + datetime.timedelta(weeks=4)
        form_data = {'renewal_date': date}
        form = RenewBookForm(data=form_data)
        self.assertTrue(form.is_valid())
</pre>

<p>前两个函数，测试字段的<code>label</code> 和 <code>help_text</code>，是否符合预期。我们必须使用字段字典访问该字段（例如<code>form.fields['renewal_date']</code>）。请注意，我们还必须测试标签值，是否为<code>None</code>，因为即使 Django 将呈现正确的标签，如果未明确设置该值，它也会返回<code>None</code>。</p>

<p>其余函数，测试表单对于续借日期，在可接受范围内是否有效，对于范围外的值，是否无效。请注意我们如何使用<code>datetime.timedelta()</code>，在当前日期（<code>datetime.date.today()</code>）周围构建测试日期值（在这种情况下指定天数或周数）。然后我们只需创建表单，传入我们的数据，并测试它是否有效。</p>

<div class="note">
<p><strong>备注：</strong> 这里我们实际上并没有使用数据库，或测试客户端。考虑修改这些测试，以使用<a href="https://docs.djangoproject.com/en/2.0/topics/testing/tools/#django.test.SimpleTestCase">SimpleTestCase</a>。</p>

<p>如果表单无效，我们还需要验证是否引发了正确的错误，但这通常是作为视图处理的一部分完成的，因此我们将在下一节中处理。</p>
</div>

<p>这就是表单的全部；我们确实有其他一些的东西，但它们是由基于类的通用编辑视图自动创建的，应该在那里进行测试！运行测试，并确认我们的代码仍然通过！</p>

<h3 id="视图">视图</h3>

<p>为了验证我们的视图行为，我们使用 <a href="https://docs.djangoproject.com/en/2.0/topics/testing/tools/#django.test.Client">Django 的测试客户端</a>。这个类，就像一个虚拟的 Web 浏览器，我们可以使用它，来模拟 URL 上的<code>GET</code>和<code>POST</code>请求，并观察响应。我们几乎可以看到，关于响应的所有内容，从低层级的 HTTP（结果标头和状态代码），到我们用来呈现 HTML 的模板，以及我们传递给它的上下文数据。我们还可以看到重定向链（如果有的话），并在每一步检查 URL，和状态代码。这允许我们验证每个视图，是否正在执行预期的操作。</p>

<p>让我们从最简单的视图开始，它提供了所有作者的列表。它显示在 URL <strong>/catalog/authors/</strong> 当中（URL 配置中，名为“authors”的 URL）。</p>

<pre class="brush: python">class AuthorListView(generic.ListView):
    model = Author
    paginate_by = 10
</pre>

<p>由于这是一个通用列表视图，几乎所有内容，都由 Django 为我们完成。可以说，如果您信任 Django，那么您唯一需要测试的，是视图可以通过正确的 URL 访问，并且可以使用其名称进行访问。但是，如果您使用的是测试驱动的开发过程，则首先编写测试，确认视图显示所有作者，并将其分成 10 个。</p>

<p>打开 <strong>/catalog/tests/test_views.py</strong> 文件，并用<code>AuthorListView</code>的以下测试代码，替换任何现有文本。和以前一样，我们导入模型，和一些有用的类。在<code>setUpTestData()</code>方法中，我们设置了许多<code>Author</code>对象，以便我们可以测试我们的分页。</p>

<pre class="brush: python">from django.test import TestCase

# Create your tests here.

from catalog.models import Author
from django.urls import reverse

class AuthorListViewTest(TestCase):

    @classmethod
    def setUpTestData(cls):
        #Create 13 authors for pagination tests
        number_of_authors = 13
        for author_num in range(number_of_authors):
            Author.objects.create(first_name='Christian %s' % author_num, last_name = 'Surname %s' % author_num,)

    def test_view_url_exists_at_desired_location(self):
        resp = self.client.get('/catalog/authors/')
        self.assertEqual(resp.status_code, 200)

    def test_view_url_accessible_by_name(self):
        resp = self.client.get(reverse('authors'))
        self.assertEqual(resp.status_code, 200)

    def test_view_uses_correct_template(self):
        resp = self.client.get(reverse('authors'))
        self.assertEqual(resp.status_code, 200)

        self.assertTemplateUsed(resp, 'catalog/author_list.html')

    def test_pagination_is_ten(self):
        resp = self.client.get(reverse('authors'))
        self.assertEqual(resp.status_code, 200)
        self.assertTrue('is_paginated' in resp.context)
        self.assertTrue(resp.context['is_paginated'] == True)
        self.assertTrue( len(resp.context['author_list']) == 10)

    def test_lists_all_authors(self):
        #Get second page and confirm it has (exactly) remaining 3 items
        resp = self.client.get(reverse('authors')+'?page=2')
        self.assertEqual(resp.status_code, 200)
        self.assertTrue('is_paginated' in resp.context)
        self.assertTrue(resp.context['is_paginated'] == True)
        self.assertTrue( len(resp.context['author_list']) == 3)</pre>

<p>所有测试，都使用客户端（属于我们的<code>TestCase</code>的派生类）来模拟<code>GET</code>请求，并获得响应（<code>resp</code>）。第一个版本检查特定 URL（注意，只是没有域名的特定路径），而第二个版本从 URL 配置中的名称生成 URL。</p>

<pre class="brush: python">resp = self.client.get('/catalog/authors/')
resp = self.client.get(reverse('authors'))
</pre>

<p>获得响应后，我们会查询其状态代码，使用的模板，响应是否已分页，返回的项目数以及项目总数。</p>

<p><br>
 我们在上面演示的最有趣的变量是<code>resp.context</code>，它是视图传递给模板的上下文变量。这对测试非常有用，因为它允许我们确认模板正在获取所需的所有数据。换句话说，我们可以检查是否正在使用预期的模板，以及模板获得的数据，这对于验证任何渲染问题，是否真的仅仅归因于模板有很大帮助。</p>

<h4 id="仅限登录用户的视图">仅限登录用户的视图</h4>

<p>在某些情况下，您需要测试仅限登录用户的视图。例如，我们的<code>LoanedBooksByUserListView</code>与我们之前的视图非常相似，但仅供登录用户使用，并且仅显示当前用户借用的<code>BookInstance</code>记录，具有出借中“on loan”状态，并且排序方式为“旧的优先”。</p>

<pre class="brush: python">from django.contrib.auth.mixins import LoginRequiredMixin

class LoanedBooksByUserListView(LoginRequiredMixin,generic.ListView):
    """
    Generic class-based view listing books on loan to current user.
    """
    model = BookInstance
    template_name ='catalog/bookinstance_list_borrowed_user.html'
    paginate_by = 10

    def get_queryset(self):
        return BookInstance.objects.filter(borrower=self.request.user).filter(status__exact='o').order_by('due_back')</pre>

<p>将以下测试代码，添加到 <strong>/catalog/tests/test_views.py</strong>。这里我们首先使用<code>SetUp()</code>创建一些用户登录帐户，和<code>BookInstance</code>对象（以及它们的相关书本，和其他记录），我们稍后将在测试中使用它们。每个测试用户都借用了一半的书本，但我们最初，将所有书本的状态设置为“维护”。我们使用了<code>SetUp()</code>而不是<code>setUpTestData()</code>，因为我们稍后会修改其中的一些对象。</p>

<div class="note">
<p><strong>备注：</strong> 下面的<code>setUp()</code>代码，会创建一个具有指定语言<code>Language</code>的书本，但您的代码可能不包含语言模型<code>Language</code>，因为它是作为挑战创建的。如果是这种情况，只需注释掉创建或导入语言对象的代码部分。您还应该在随后的<code>RenewBookInstancesViewTest</code>部分中，执行此操作。</p>
</div>

<pre class="brush: python">import datetime
from django.utils import timezone

from catalog.models import BookInstance, Book, Genre, Language
from django.contrib.auth.models import User #Required to assign User as a borrower

class LoanedBookInstancesByUserListViewTest(TestCase):

    def setUp(self):
        #Create two users
        test_user1 = User.objects.create_user(username='testuser1', password='12345')
        test_user1.save()
        test_user2 = User.objects.create_user(username='testuser2', password='12345')
        test_user2.save()

        #Create a book
        test_author = Author.objects.create(first_name='John', last_name='Smith')
        test_genre = Genre.objects.create(name='Fantasy')
        test_language = Language.objects.create(name='English')
        test_book = Book.objects.create(title='Book Title', summary = 'My book summary', isbn='ABCDEFG', author=test_author, language=test_language)
        # Create genre as a post-step
        genre_objects_for_book = Genre.objects.all()
        test_book.genre.set(genre_objects_for_book) #Direct assignment of many-to-many types not allowed.
        test_book.save()

        #Create 30 BookInstance objects
        number_of_book_copies = 30
        for book_copy in range(number_of_book_copies):
            return_date= timezone.now() + datetime.timedelta(days=book_copy%5)
            if book_copy % 2:
                the_borrower=test_user1
            else:
                the_borrower=test_user2
            status='m'
            BookInstance.objects.create(book=test_book,imprint='Unlikely Imprint, 2016', due_back=return_date, borrower=the_borrower, status=status)

    def test_redirect_if_not_logged_in(self):
        resp = self.client.get(reverse('my-borrowed'))
        self.assertRedirects(resp, '/accounts/login/?next=/catalog/mybooks/')

    def test_logged_in_uses_correct_template(self):
        login = self.client.login(username='testuser1', password='12345')
        resp = self.client.get(reverse('my-borrowed'))

        #Check our user is logged in
        self.assertEqual(str(resp.context['user']), 'testuser1')
        #Check that we got a response "success"
        self.assertEqual(resp.status_code, 200)

        #Check we used correct template
        self.assertTemplateUsed(resp, 'catalog/bookinstance_list_borrowed_user.html')
</pre>

<p>要验证如果用户未登录，视图将重定向到登录页面，我们使用<code>assertRedirects</code>，如<code>test_redirect_if_not_logged_in()</code>中所示。要验证是否已为登录用户显示该页面，我们首先登录我们的测试用户，然后再次访问该页面，并检查我们获得的<code>status_code</code>为 200（成功）。</p>

<p>测试的其余部分，验证我们的观点，仅返回借给当前借用人的书本。复制上面测试类末尾的（自解释）代码。</p>

<pre class="brush: python">    def test_only_borrowed_books_in_list(self):
        login = self.client.login(username='testuser1', password='12345')
        resp = self.client.get(reverse('my-borrowed'))

        #Check our user is logged in
        self.assertEqual(str(resp.context['user']), 'testuser1')
        #Check that we got a response "success"
        self.assertEqual(resp.status_code, 200)

        #Check that initially we don't have any books in list (none on loan)
        self.assertTrue('bookinstance_list' in resp.context)
        self.assertEqual( len(resp.context['bookinstance_list']),0)

        #Now change all books to be on loan
        get_ten_books = BookInstance.objects.all()[:10]

        for copy in get_ten_books:
            copy.status='o'
            copy.save()

        #Check that now we have borrowed books in the list
        resp = self.client.get(reverse('my-borrowed'))
        #Check our user is logged in
        self.assertEqual(str(resp.context['user']), 'testuser1')
        #Check that we got a response "success"
        self.assertEqual(resp.status_code, 200)

        self.assertTrue('bookinstance_list' in resp.context)

        #Confirm all books belong to testuser1 and are on loan
        for bookitem in resp.context['bookinstance_list']:
            self.assertEqual(resp.context['user'], bookitem.borrower)
            self.assertEqual('o', bookitem.status)

    def test_pages_ordered_by_due_date(self):

        #Change all books to be on loan
        for copy in BookInstance.objects.all():
            copy.status='o'
            copy.save()

        login = self.client.login(username='testuser1', password='12345')
        resp = self.client.get(reverse('my-borrowed'))

        #Check our user is logged in
        self.assertEqual(str(resp.context['user']), 'testuser1')
        #Check that we got a response "success"
        self.assertEqual(resp.status_code, 200)

        #Confirm that of the items, only 10 are displayed due to pagination.
        self.assertEqual( len(resp.context['bookinstance_list']),10)

        last_date=0
        for copy in resp.context['bookinstance_list']:
            if last_date==0:
                last_date=copy.due_back
            else:
                self.assertTrue(last_date &lt;= copy.due_back)</pre>

<p>你也可以添加分页测试，如果你愿意的话！</p>

<h4 id="使用表单测试视图">使用表单测试视图</h4>

<p>使用表单测试视图，比上面的情况稍微复杂一些，因为您需要测试更多代码路径：初始显示，数据验证失败后显示，以及验证成功后显示。好消息是，我们使用客户端进行测试的方式，与我们对仅显示视图的方式，几乎完全相同。</p>

<p>为了演示，让我们为用于续借书本的视图，编写一些测试（<code>renew_book_librarian()</code>）：</p>

<pre class="brush: python">from .forms import RenewBookForm

@permission_required('catalog.can_mark_returned')
def renew_book_librarian(request, pk):
    """
    View function for renewing a specific BookInstance by librarian
    """
    book_inst=get_object_or_404(BookInstance, pk = pk)

    # If this is a POST request then process the Form data
    if request.method == 'POST':

        # Create a form instance and populate it with data from the request (binding):
        form = RenewBookForm(request.POST)

        # Check if the form is valid:
        if form.is_valid():
            # process the data in form.cleaned_data as required (here we just write it to the model due_back field)
            book_inst.due_back = form.cleaned_data['renewal_date']
            book_inst.save()

            # redirect to a new URL:
            return HttpResponseRedirect(reverse('all-borrowed') )

    # If this is a GET (or any other method) create the default form
    else:
        proposed_renewal_date = datetime.date.today() + datetime.timedelta(weeks=3)
        form = RenewBookForm(initial={'renewal_date': proposed_renewal_date,})

    return render(request, 'catalog/book_renew_librarian.html', {'form': form, 'bookinst':book_inst})</pre>

<p>我们需要测试该视图，仅供具有<code>can_mark_returned</code>权限的用户使用，并且如果用户尝试续借不存在的<code>BookInstance</code>，则会将用户重定向到 HTTP 404 错误页面。我们应该检查表单的初始值，是否以未来三周的日期为参考值，如果验证成功，我们将被重定向到“所有借阅的书本”视图。作为验证 - 失败测试的一部分，我们还将检查我们的表单，是否发送了相应的错误消息。</p>

<p>将测试类的第一部分（如下所示），添加到 <strong>/catalog/tests/test_views.py</strong> 的底部。这将创建两个用户和两个书本实例，但只为一个用户提供访问该视图所需的权限。在测试期间，授予权限的代码以<strong>粗体</strong>显示：</p>

<pre class="brush: python">from django.contrib.auth.models import Permission # Required to grant the permission needed to set a book as returned.

class RenewBookInstancesViewTest(TestCase):

    def setUp(self):
        #Create a user
        test_user1 = User.objects.create_user(username='testuser1', password='12345')
        test_user1.save()

        test_user2 = User.objects.create_user(username='testuser2', password='12345')
        test_user2.save()
<strong>        permission = Permission.objects.get(name='Set book as returned')
        test_user2.user_permissions.add(permission)
        test_user2.save()</strong>

        #Create a book
        test_author = Author.objects.create(first_name='John', last_name='Smith')
        test_genre = Genre.objects.create(name='Fantasy')
        test_language = Language.objects.create(name='English')
        test_book = Book.objects.create(title='Book Title', summary = 'My book summary', isbn='ABCDEFG', author=test_author, language=test_language,)
        # Create genre as a post-step
        genre_objects_for_book = Genre.objects.all()
        test_book.genre.set(genre_objects_for_book) # Direct assignment of many-to-many types not allowed.
        test_book.save()

        #Create a BookInstance object for test_user1
        return_date= datetime.date.today() + datetime.timedelta(days=5)
        self.test_bookinstance1=BookInstance.objects.create(book=test_book,imprint='Unlikely Imprint, 2016', due_back=return_date, borrower=test_user1, status='o')

        #Create a BookInstance object for test_user2
        return_date= datetime.date.today() + datetime.timedelta(days=5)
        self.test_bookinstance2=BookInstance.objects.create(book=test_book,imprint='Unlikely Imprint, 2016', due_back=return_date, borrower=test_user2, status='o')</pre>

<p>将以下测试添加到测试类的底部。这些检查只有具有正确权限的用户（testuser2）才能访问该视图。我们检查所有情况：当用户没有登录时、当用户登录但没有正确的权限，当用户有权限但不是借用人（应该成功），以及当他们尝试访问不存在的<code>BookInstance</code>，会发生什么。我们还检查是否使用了正确的模板。</p>

<pre class="brush: python">    def test_redirect_if_not_logged_in(self):
        resp = self.client.get(reverse('renew-book-librarian', kwargs={'pk':self.test_bookinstance1.pk,}) )
        #Manually check redirect (Can't use assertRedirect, because the redirect URL is unpredictable)
        self.assertEqual( resp.status_code,302)
        self.assertTrue( resp.url.startswith('/accounts/login/') )

    def test_redirect_if_logged_in_but_not_correct_permission(self):
        login = self.client.login(username='testuser1', password='12345')
        resp = self.client.get(reverse('renew-book-librarian', kwargs={'pk':self.test_bookinstance1.pk,}) )

        #Manually check redirect (Can't use assertRedirect, because the redirect URL is unpredictable)
        self.assertEqual( resp.status_code,302)
        self.assertTrue( resp.url.startswith('/accounts/login/') )

    def test_logged_in_with_permission_borrowed_book(self):
        login = self.client.login(username='testuser2', password='12345')
        resp = self.client.get(reverse('renew-book-librarian', kwargs={'pk':self.test_bookinstance2.pk,}) )

        #Check that it lets us login - this is our book and we have the right permissions.
        self.assertEqual( resp.status_code,200)

    def test_logged_in_with_permission_another_users_borrowed_book(self):
        login = self.client.login(username='testuser2', password='12345')
        resp = self.client.get(reverse('renew-book-librarian', kwargs={'pk':self.test_bookinstance1.pk,}) )

        #Check that it lets us login. We're a librarian, so we can view any users book
        self.assertEqual( resp.status_code,200)

    def test_HTTP404_for_invalid_book_if_logged_in(self):
        import uuid
        test_uid = uuid.uuid4() #unlikely UID to match our bookinstance!
        login = self.client.login(username='testuser2', password='12345')
        resp = self.client.get(reverse('renew-book-librarian', kwargs={'pk':test_uid,}) )
        self.assertEqual( resp.status_code,404)

    def test_uses_correct_template(self):
        login = self.client.login(username='testuser2', password='12345')
        resp = self.client.get(reverse('renew-book-librarian', kwargs={'pk':self.test_bookinstance1.pk,}) )
        self.assertEqual( resp.status_code,200)

        #Check we used correct template
        self.assertTemplateUsed(resp, 'catalog/book_renew_librarian.html')
</pre>

<p>添加下一个测试方法，如下所示。这将检查表单的初始日期，是将来三周。请注意我们如何能够访问表单字段的初始值内的值（以<strong>粗体</strong>显示）。</p>

<pre class="brush: python">    def test_form_renewal_date_initially_has_date_three_weeks_in_future(self):
        login = self.client.login(username='testuser2', password='12345')
        resp = self.client.get(reverse('renew-book-librarian', kwargs={'pk':self.test_bookinstance1.pk,}) )
        self.assertEqual( resp.status_code,200)

        date_3_weeks_in_future = datetime.date.today() + datetime.timedelta(weeks=3)
        self.assertEqual(<strong>resp.context['form'].initial['renewal_date']</strong>, date_3_weeks_in_future )
</pre>

<p>下一个测试（将其添加到类中）会检查如果续借成功，视图会重定向到所有借书的列表。这里的不同之处在于，我们首次展示了，如何使用客户端发布（<code>POST</code>）数据。post 数据是 post 函数的第二个参数，并被指定为键/值的字典。</p>

<pre class="brush: python">    def test_redirects_to_all_borrowed_book_list_on_success(self):
        login = self.client.login(username='testuser2', password='12345')
        valid_date_in_future = datetime.date.today() + datetime.timedelta(weeks=2)
        resp = <strong>self.client.<em>post</em>(reverse('renew-book-librarian', kwargs={'pk':self.test_bookinstance1.pk,}), {'renewal_date':valid_date_in_future} )</strong>
        self.assertRedirects(resp, reverse('all-borrowed') )
</pre>

<div class="warning">
<p><strong>警告：</strong> 全部借用的视图作为额外挑战，您的代码可能会改为重定向到主页'/'。如果是这样，请将测试代码的最后两行，修改为与下面的代码类似。请求中的<code>follow=True</code>，确保请求返回最终目标 URL（因此检查<code>/catalog/</code>而不是<code>/</code>）。</p>

<pre class="brush: python"> resp = self.client.post(reverse('renew-book-librarian', kwargs={'pk':self.test_bookinstance1.pk,}), {'renewal_date':valid_date_in_future},<strong>follow=True</strong> )
 <strong>self.assertRedirects(resp, '/catalog/')</strong></pre>
</div>

<p>将最后两个函数，复制到类中，如下所示。这些再次测试<code>POST</code>请求，但在这种情况下具有无效的续借日期。我们使用<code>assertFormError() </code>，来验证错误消息是否符合预期。</p>

<pre class="brush: python">    def test_form_invalid_renewal_date_past(self):
        login = self.client.login(username='testuser2', password='12345')
        date_in_past = datetime.date.today() - datetime.timedelta(weeks=1)
        resp = self.client.post(reverse('renew-book-librarian', kwargs={'pk':self.test_bookinstance1.pk,}), {'renewal_date':date_in_past} )
        self.assertEqual( resp.status_code,200)
        <strong>self.assertFormError(resp, 'form', 'renewal_date', 'Invalid date - renewal in past')</strong>

    def test_form_invalid_renewal_date_future(self):
        login = self.client.login(username='testuser2', password='12345')
        invalid_date_in_future = datetime.date.today() + datetime.timedelta(weeks=5)
        resp = self.client.post(reverse('renew-book-librarian', kwargs={'pk':self.test_bookinstance1.pk,}), {'renewal_date':invalid_date_in_future} )
        self.assertEqual( resp.status_code,200)
        <strong>self.assertFormError(resp, 'form', 'renewal_date', 'Invalid date - renewal more than 4 weeks ahead')</strong>
</pre>

<p>可以使用相似的技术，来测试其他视图。</p>

<h3 id="模板">模板</h3>

<p>Django 提供测试 API 来检查您的视图，是否正在调用正确的模板，并允许您验证，是否正在发送正确的信息。但是，没有特定的 API，支持在 Django 中测试 HTML 输出，是否按预期呈现。</p>

<h2 id="其他推荐的测试工具">其他推荐的测试工具</h2>

<p>Django 的测试框架，可以帮助您编写有效的单元和集成测试 - 我们只涉及底层单元测试框架<strong>unittest</strong>可以做什么，而不去谈 Django 的其他部分（例如，查看如何使用<a href="https://docs.python.org/3.5/library/unittest.mock-examples.html">unittest.mock</a> 修补第三方库，以便您可以更彻底地测试自己的代码）。</p>

<p>虽然您可以使用许多其他测试工具，但我们只重点介绍两个：</p>

<ul>
 <li><a href="http://coverage.readthedocs.io/en/latest/">Coverage</a>: 此 Python 工具报告您的测试，实际执行了多少代码。当开始使用时，你正试图找出你应该测试的确切内容，它会特别有用。</li>
 <li><a href="/en-US/docs/Learn/Tools_and_testing/Cross_browser_testing/Your_own_automation_environment">Selenium</a> 是一个在真实浏览器中，自动化测试的框架。它允许您模拟与站点交互的真实用户，并为系统测试您的站点，提供了一个很好的框架（从集成测试开始的下一步）。</li>
</ul>

<h2 id="挑战自己">挑战自己</h2>

<p>有许多模型与视图，我们可以用来测试。比如一个简单的任务，试着为<code>AuthorCreate</code>视图，创造一个测试案例。</p>

<pre class="brush: python">class AuthorCreate(PermissionRequiredMixin, CreateView):
    model = Author
    fields = '__all__'
    initial={'date_of_death':'12/10/2016',}
    permission_required = 'catalog.can_mark_returned'</pre>

<p>请记住，您需要检查您指定的任何内容、或设计的一部分。这将包括谁有权访问，初始日期，使用的模板，以及视图在成功时，重定向的位置。</p>

<h2 id="总结">总结</h2>

<p>撰写测试代码既不有趣也不吸引人，因此在创造一个网站时，经常被留到最后才处理（或者完全不处理）。然而，它是一个基础的部分，以保证你的程式码，在更改之后是安全、可发布的，并且维护起来不会花费太多成本。</p>

<p>本教程中，我們演示了如何为模型、表单和视图，编写并运行测试。最重要的是，我们已经提供给您，应该测试的内容的简短摘要，这通常是您开始时，最难解决的问题。还有很多东西要知道，但即使你已经学到了什么，你也应该能够为你的网站创建有效的单元测试。</p>

<p>下一个、也是最后一个教程，展示了如何部署精彩的（并经过全面测试的！）Django 网站。</p>

<h2 id="也可以参考">也可以参考</h2>

<ul>
 <li><a href="https://docs.djangoproject.com/en/2.0/topics/testing/overview/">Writing and running tests</a> (Django docs)</li>
 <li><a href="https://docs.djangoproject.com/en/2.0/intro/tutorial05/">Writing your first Django app, part 5 &gt; Introducing automated testing</a> (Django docs)</li>
 <li><a href="https://docs.djangoproject.com/en/2.0/topics/testing/tools/">Testing tools reference</a> (Django docs)</li>
 <li><a href="https://docs.djangoproject.com/en/2.0/topics/testing/advanced/">Advanced testing topics</a> (Django docs)</li>
 <li><a href="http://toastdriven.com/blog/2011/apr/10/guide-to-testing-in-django/">A Guide to Testing in Django</a> (Toast Driven Blog, 2011)</li>
 <li><a href="http://test-driven-django-development.readthedocs.io/en/latest/index.html">Workshop: Test-Driven Web Development with Django</a> (San Diego Python, 2014)</li>
 <li><a href="https://realpython.com/blog/python/testing-in-django-part-1-best-practices-and-examples/">Testing in Django (Part 1) - Best Practices and Examples</a> (RealPython, 2013)</li>
</ul>

<p>{{PreviousMenuNext("Learn/Server-side/Django/Forms", "Learn/Server-side/Django/Deployment", "Learn/Server-side/Django")}}</p>

<p> </p>

<h2 id="本系列教程">本系列教程</h2>

<ul>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Introduction">Django introduction</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/development_environment">Setting up a Django development environment</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Tutorial_local_library_website">Django Tutorial: The Local Library website</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/skeleton_website">Django Tutorial Part 2: Creating a skeleton website</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Models">Django Tutorial Part 3: Using models</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Admin_site">Django Tutorial Part 4: Django admin site</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Home_page">Django Tutorial Part 5: Creating our home page</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Generic_views">Django Tutorial Part 6: Generic list and detail views</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Sessions">Django Tutorial Part 7: Sessions framework</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Authentication">Django Tutorial Part 8: User authentication and permissions</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Forms">Django Tutorial Part 9: Working with forms</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Testing">Django Tutorial Part 10: Testing a Django web application</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Deployment">Django Tutorial Part 11: Deploying Django to production</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/web_application_security">Django web application security</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/django_assessment_blog">DIY Django mini blog</a></li>
</ul>

<p> </p>