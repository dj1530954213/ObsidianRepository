TestTaskManager中需要实现以下的功能：
1. 根据我们项目中的ChannelMapping集合内部的信息为每一个测试点位创建测试Task(需要区分AI、AO、DI、DO)创建不同的测试任务，且将测试任务通过一些方法来管理开始、暂停、添加、删除等操作，这个任务的内部我们稍后再写入具体实现，你先帮我创建好ITestTaskManager的接口内的方法。要求如下：
	1. 测试任务之间不能相互干扰，且传入的和PLC的通讯实例对于测试PLC和被测PLC都是单例的。
	2. 对于单点测试任务需要能同步开展，使用多线程来实现。
	3. 线程内部由于需要按照流程进行测试所以需要等待固定的时间接收测试结果。所以测试线程不能阻塞主线程。一般一个AI、AO点位需要测试15秒左右
	4. 我考虑如果所有的点位都单独开一个线程那么线程池肯定不够，我希望你帮我设计好Task的处理方式，能保证所有单点测试同步开展并且不会造成频繁的上下文切换从而影响性能

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.ComponentModel;

namespace FatFullVersion.Models
{
    /// <summary>
    /// 测试结果类，用于存储测试的结果数据
    /// </summary>
    public class TestResult : INotifyPropertyChanged
    {
        private string _variableName;
        /// <summary>
        /// 变量名称
        /// </summary>
        public string VariableName
        {
            get { return _variableName; }
            set
            {
                if (_variableName != value)
                {
                    _variableName = value;
                    OnPropertyChanged(nameof(VariableName));
                }
            }
        }

        private string _testPlcChannel;
        /// <summary>
        /// 测试PLC通道标识
        /// </summary>
        public string TestPlcChannel
        {
            get { return _testPlcChannel; }
            set
            {
                if (_testPlcChannel != value)
                {
                    _testPlcChannel = value;
                    OnPropertyChanged(nameof(TestPlcChannel));
                }
            }
        }

        private DateTime _startTime;
        /// <summary>
        /// 测试开始时间
        /// </summary>
        public DateTime StartTime
        {
            get { return _startTime; }
            set
            {
                if (_startTime != value)
                {
                    _startTime = value;
                    OnPropertyChanged(nameof(StartTime));
                }
            }
        }

        private DateTime _endTime;
        /// <summary>
        /// 测试结束时间
        /// </summary>
        public DateTime EndTime
        {
            get { return _endTime; }
            set
            {
                if (_endTime != value)
                {
                    _endTime = value;
                    OnPropertyChanged(nameof(EndTime));
                    // 当结束时间设置时，自动计算测试持续时间
                    OnPropertyChanged(nameof(TestDuration));
                }
            }
        }

        /// <summary>
        /// 测试持续时间（秒）
        /// </summary>
        public double TestDuration
        {
            get
            {
                if (_endTime > DateTime.MinValue && _startTime > DateTime.MinValue)
                {
                    return (_endTime - _startTime).TotalSeconds;
                }
                return 0;
            }
        }

        private double _expectedValue;
        /// <summary>
        /// 期望值
        /// </summary>
        public double ExpectedValue
        {
            get { return _expectedValue; }
            set
            {
                if (_expectedValue != value)
                {
                    _expectedValue = value;
                    OnPropertyChanged(nameof(ExpectedValue));
                }
            }
        }

        private double _actualValue;
        /// <summary>
        /// 实际值
        /// </summary>
        public double ActualValue
        {
            get { return _actualValue; }
            set
            {
                if (_actualValue != value)
                {
                    _actualValue = value;
                    OnPropertyChanged(nameof(ActualValue));
                    // 当实际值设置时，自动计算偏差值
                    OnPropertyChanged(nameof(Deviation));
                    OnPropertyChanged(nameof(DeviationPercent));
                }
            }
        }

        /// <summary>
        /// 偏差值
        /// </summary>
        public double Deviation
        {
            get { return ActualValue - ExpectedValue; }
        }

        /// <summary>
        /// 偏差百分比
        /// </summary>
        public double DeviationPercent
        {
            get
            {
                if (ExpectedValue != 0)
                {
                    return (Deviation / ExpectedValue) * 100;
                }
                return 0;
            }
        }

        private string _status;
        /// <summary>
        /// 测试状态（通过/失败/取消等）
        /// </summary>
        public string Status
        {
            get { return _status; }
            set
            {
                if (_status != value)
                {
                    _status = value;
                    OnPropertyChanged(nameof(Status));
                }
            }
        }

        private string _errorMessage;
        /// <summary>
        /// 错误信息
        /// </summary>
        public string ErrorMessage
        {
            get { return _errorMessage; }
            set
            {
                if (_errorMessage != value)
                {
                    _errorMessage = value;
                    OnPropertyChanged(nameof(ErrorMessage));
                }
            }
        }

        private string _batchName;
        /// <summary>
        /// 测试批次名称
        /// </summary>
        public string BatchName
        {
            get { return _batchName; }
            set
            {
                if (_batchName != value)
                {
                    _batchName = value;
                    OnPropertyChanged(nameof(BatchName));
                }
            }
        }

        public event PropertyChangedEventHandler PropertyChanged;

        protected virtual void OnPropertyChanged(string propertyName)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
    }
} 

```


我现在需要你帮我完成以下的这些工作：
修改批次选择弹窗中绑定的相关数据：
首先我先定义需要的列如下：
批次名称、测试项点数、创建时间、初次测试时间、最终测试时间、测试状态
其次我需要用户在导入点表后，系统生成默认的测试批次的同时将批次信息同步的更新到这个批次选择窗口中，其中
创建时间为自动生成点表的时间后续需要能通过其他的时间进行修改
测试项点数为当前批次4中类型点数的总和
初次测试时间为同组内测试时间最早初次的测试时间为准，最终测试时间为同组内最终测试时间最晚的为准。测试状态已完成、取消、进行中、未开始这四种状态为准。请你在修改前端界面(DataEditView)的同时帮我修改后台逻辑中涉及到的数据模型和处理逻辑


现在让我们来解决下面的这些问题
1、批次选择窗口中的批次状态没有根据批次内的测试点状态实时更新
2、测试队列显示区域
3、AO点位的低低报、低报、高报、高高报在测试的时候不涉及调整为NaN
4、AI点位在测试的时候高报高高报区域点击确认通过后对于的测试结果中的高报反馈状态、高高报反馈状态的结果都设置为已通过。低报及低低报区域也同理
5、测试结果中的内容设置为追加模式，对于同样的内容替换，不同的内容追加。结果尽可能的详细

请你在修改完之后编译确定没有错误



现在让我们来解决一些bug：
1、最初导入点表的时候测试队列显示区域和测试结果区域的批次号都显示正常。但是当选择完批次后点击完成接线确认时测试队列显示区域的测试批次就变为了测试ID，当第二次选择批次的时候测试结果队列中测试批次变为了另一组测试ID。我觉得在我的程序中只需要测试批次名而不需要测试ID。且在导入点表之后所有点位的测试批次在任何时候都不应该发生变化。
2、通道的硬点自动测试结果并没有同步的更新至测试点位集合AllChannels的硬点测试结果中，界面也未同步显示测试结果
3、在我的设计中测试结果区域中应该像测试队列显示区域一样显示通道硬点测试结果。在最终的测试结果中也应该有通道硬点的测试结果。且AI、AO点位的测试过程值也应该显示在测试结果区域的0%~100%的结果中。
DI和DO点位的通道硬点测试应该更新在测试结果中

我希望你先查看一遍代码确认你已经了解了项目的整体逻辑再进行代码的修改。修改完成之后进行编译确保没有任何错误



现在请你修复如下的bug
1、在我们的测试中分为通道自动测试与手动测试两个部分。当通道测试完成之后我们测试结果区域的最后一列中不应该显示的是未测试而是通道硬点测试的结果。且批次选择窗口中的批次测试状态也不应该是未测试而是测试中。当自动测试与手动测试也完成之后测试结果区域中的行应该变为绿色，当其中任意一个发生错误的时候都应该是我们先前定义的红色从而提醒测试者。

请你先阅读相关代码，确认了解了整个逻辑后再修改代码，修改完成后编译一下代码确保代码没有错误或者逻辑上的错误。请确保一次修复上面提到的bug
![[Pasted image 20250329182117.png]]

1、首先测试状态已经能在测试结果区域的最终测试结果终显示。但是我觉得描述不正确，描述应该为硬点通道测试成功或者是硬点通道测试失败并显示失败原因，且从整个测试流程中来说，这个最终的结果应该不是替换整体的结果而是附加在这个最终测试的结果中。
2、测试批次选择窗口中依然现实的是未测试。我觉得这个批次测试状态应该显示的状态如下：
1. 未进行任何测试的时候显示未测试。
2. 测试完硬点通道测试的时候如果失败显示硬点通道测试失败请重新测试，如果成功显示硬点测试通过请开始手动测试
3. 硬点通道测试与手动测试都通过后显示测试完成
请你先阅读相关代码，确认了解了整个逻辑后再修改代码，修改完成后编译一下代码确保代码没有错误或者逻辑上的错误。请确保一次修复上面提到的bug


我需要你查看我的所有代码了解所有的逻辑关系。
现在让我们来解决所有的bug：
1、我希望你帮我修改AITestTask、AOTestTask、DITestTask、DOTestTask和private async void StartTest()的有关错误日志记录的代码满足如下要求：
通道硬点测试结果中显示测试过程中的详细信息。通道硬点测试完成后总的测试结果中只显示通道硬点测试成功或者失败。注意。不要修改测试任务中的通讯相关的东西。只修改信息记录相关的
2、手动测试窗口中的所有按钮点击都不要弹出提示对话框
3、通道硬点测试完成后批次选择框中应该为测试中。当手动测试也完成后显示为全部通过。


我发现了一些问题。
1、当硬点通道测试有结果时会影响手动测试窗口中的按钮的颜色。实际上这两部分应该毫无关联。
2、手动测试窗口中的AO、DI、DO点击后测试结果区域的状态不实时更新。行没有变绿，批次选择窗口在硬点通道测试完成后依然显示未测试，实际应该显示测试中。当手动测试同过后应该显示测试完成。


让我们重新梳理一下测试的流程和要求：
1、用户选择完批次之后点击完成接线确认，通道硬点自动测试此时可以点击。
2、用户点击完通道硬点自动测试后开始自动测试，测试的详细结果保存在ChannelMapping的HardPointTestResult中，如果硬点测试的结果为成功则ResultText中显示硬点通道测试成功。如果失败则HardPointTestResult中显示详细的异常信息，而ResultText中只显示硬点通道测试失败。
3、当用户解决完所有的硬点通道问题后开始进行手动测试环节。
4、用户点击每一个测试项的打开手动测试窗口后。对应的手动测试窗口显示的确认通过按钮的颜色与ChannelMapping中对应的属性关联：

AI点位LowAlarmStatus和LowLowAlarmStatus与低报及低低报核对区域内的确认通过按钮关联、HighAlarmStatus和HighHighAlarmStatus与高报及高高报核对区域内的确认通过按钮关联。MaintenanceFunction与维护功能核对的确认通过按钮关联，显示值核对需要你帮我新加一个名叫ShowValueStatus属性来存储状态。

AO点的手动测试窗口只有一个确认通过按钮其状态与ShowValueStatus进行绑定

DI点的手动测试窗口只有一个确认通过按钮其状态与ShowValueStatus进行绑定

DO点的手动测试窗口只有一个确认通过按钮其状态与ShowValueStatus进行绑定

这些状态初始都为false。当用户点击确认通过的时候为true。
5、当硬点通道测试与手动测试环节全部通过。更新结果显示区域的最终测试结果。使其显示为绿色

请你根据我的要求更改我的代码帮我完成我需要的功能。请一次完成


现在存在几个问题：
1、结果显示区域的这个点位汇总信息不能实时更新，比如待测试点位、成功点位、已测试点位、失败点位的数量。只有在测试完再次打开批次选择窗口的时候才会更新，我需要它能实时更新。
2、测试结果区域中的最后一列的测试结果需要在开始手动测试的时候更新为手动测试中。当这个点位完全彻底通过的时候显示为已通过。




